<template>
    <!-- Titolo della pagina -->
    <div class="text-h2">WORLD RADIO</div><br>
    <!-- Barra di navigazione -->
    <div class="navbar" v-if="selectedRadio">
        <!-- Logo della radio -->
        <img :src="getRadioImage(selectedRadio)" class="radio-logo" alt="Radio logo">
        <!-- Nome della radio -->
        <h4>{{ selectedRadio.name }}</h4>
        <!-- Paese della radio -->
        <h3>{{ selectedRadio.country }}</h3>
        <!-- Pulsante per avviare o interrompere la riproduzione -->
        <v-btn :icon="selectedRadio.playing ? 'mdi-stop' : 'mdi-play'" @click="togglePlayPause(selectedRadio)"></v-btn>
    </div>
    <!-- Contenitore per il rendering della scena 3D -->
    <div ref="container"></div>
</template>

<script>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import earthTexture from '../assets/8k_earth_daymap.jpg';
import Hls from 'hls.js';
import defaultImage from '../assets/radio.png';

export default {
    name: 'WorldView',
    data() {
        return {
            camera: null,
            renderer: null,
            controls: null,
            earthRadius: 1,
            radio: [],
            selectedRadio: null,
            defaultImage,
            isFavorite: false
        };
    },
    mounted() {
        // Inizializza la scena 3D
        this.init();
        // Avvia l'animazione
        this.animate();
        // Ottiene le radio
        this.getRadios();
        // Aggiunge eventi per il ridimensionamento della finestra e per il click del mouse
        window.addEventListener('resize', this.handleWindowResize);
        window.addEventListener('click', this.onDocumentMouseClick);
    },
    beforeUnmount() {
        // Rimuove gli event listener prima della disattivazione del componente
        window.removeEventListener('resize', this.handleWindowResize);
        window.removeEventListener('click', this.onDocumentMouseClick);
    },
    methods: {
        // Inizializza la scena 3D
        init() {
            // Inizializza la camera
            this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            this.camera.position.z = 2;

            // Inizializza il renderer e imposta le dimensioni
            this.renderer = new THREE.WebGLRenderer();
            const width = window.innerWidth * 0.96; // Imposta la larghezza al 80% della finestra del browser
            const height = window.innerHeight * 0.70; // Imposta l'altezza al 60% della finestra del browser
            this.renderer.setSize(width, height);
            this.$refs.container.appendChild(this.renderer.domElement);

            // Inizializza i controlli della scena
            this.controls = new OrbitControls(this.camera, this.renderer.domElement);
            this.controls.enableDamping = true;

            // Inizializza la scena
            const scene = new THREE.Scene();

            // Aggiunge la Terra alla scena
            const geometry = new THREE.SphereGeometry(this.earthRadius, 64, 64); // Aumenta i segmenti per una superficie più liscia
            const texture = new THREE.TextureLoader().load(earthTexture);
            const material = new THREE.MeshPhongMaterial({ map: texture });
            const earth = new THREE.Mesh(geometry, material);
            scene.add(earth);

            // Aggiunge luci alla scena
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
            scene.add(ambientLight);
            const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
            directionalLight.position.set(1, 1, 1).normalize();
            scene.add(directionalLight);

            // Imposta la scena
            this.scene = scene;

            // Inizializza il raycaster e il vettore del mouse
            this.raycaster = new THREE.Raycaster();
            this.mouse = new THREE.Vector2();
        },
        // Gestisce il click del mouse sulla scena
        onDocumentMouseClick(event) {
            // Impedisce l'azione predefinita del click
            event.preventDefault();
            // Calcola la posizione del mouse rispetto alla finestra
            this.mouse.x = (event.offsetX / this.renderer.domElement.clientWidth) * 2 - 1;
            this.mouse.y = -(event.offsetY / this.renderer.domElement.clientHeight) * 2 + 1;
            // Imposta il raycaster e interseca gli oggetti nella scena
            this.raycaster.setFromCamera(this.mouse, this.camera);
            const intersects = this.raycaster.intersectObjects(this.scene.children);

            // Se ci sono intersezioni
            if (intersects.length > 0) {
                // Gestisce il click sull'oggetto intersecato
                this.handleMarkerClick(intersects[0]);
            }
        },
        // Gestisce il click su un marker nella scena 3D
        handleMarkerClick(intersectedObject) {
            if (intersectedObject.object.onClick) {
                intersectedObject.object.onClick();
                const previousRadio = this.selectedRadio;
                this.selectedRadio = intersectedObject.object.userData;
                this.$nextTick(() => {
                    if (this.selectedRadio.url_resolved || this.selectedRadio.url) {
                        this.lazyLoadAudio(this.selectedRadio);
                        if (previousRadio && previousRadio !== this.selectedRadio) {
                            this.pauseRadio(previousRadio);
                        }
                    }
                });
            }
        },
        // Carica in modo pigro l'audio per la radio selezionata
        lazyLoadAudio(radio) {
            // Verifica se l'audioPlayer esiste già
            if (!radio.audioPlayer) {
                // Crea un nuovo audio player se non esiste
                radio.audioPlayer = new Audio(radio.url_resolved || radio.url);
                radio.audioPlayer.onloadeddata = () => {
                    console.log('Audio loaded');
                };
                radio.audioPlayer.onerror = () => {
                    console.error('Error loading audio');
                };
            }
        },
        // Aggiunge un marker alla scena 3D
        addMarker(longitude, latitude, radio) {
            // Calcola le coordinate 3D dal longitudine e latitudine
            if (longitude !== null && latitude !== null) {
                const phi = (90 - latitude) * (Math.PI / 180);
                const theta = (longitude + 180) * (Math.PI / 180);
                const x = -this.earthRadius * Math.sin(phi) * Math.cos(theta);
                const y = this.earthRadius * Math.cos(phi);
                const z = this.earthRadius * Math.sin(phi) * Math.sin(theta);

                // Crea il marker visibile
                const geometry = new THREE.SphereGeometry(0.01, 32, 32);
                const material = new THREE.MeshBasicMaterial({ color: 0xff0000 });
                const marker = new THREE.Mesh(geometry, material);
                marker.position.set(x, y, z);
                marker.userData = radio;
                marker.onClick = () => {
                    console.log('Clicked on marker:', marker.userData);
                };
                this.scene.add(marker);

                // Crea la sfera invisibile per migliorare l'interazione
                const invisibleSphereGeometry = new THREE.SphereGeometry(0.015, 32, 32);
                const invisibleSphereMaterial = new THREE.MeshBasicMaterial({ visible: false });
                const invisibleSphere = new THREE.Mesh(invisibleSphereGeometry, invisibleSphereMaterial);
                invisibleSphere.position.set(x, y, z);
                invisibleSphere.userData = radio;
                invisibleSphere.onClick = marker.onClick;
                this.scene.add(invisibleSphere);
            } else {
                console.error('Longitude or latitude is null');
            }
        },
        // Animazione della scena 3D
        animate() {
            requestAnimationFrame(this.animate);

            if (this.controls) {
                this.controls.update();
            }

            if (this.renderer && this.scene && this.camera) {
                this.renderer.render(this.scene, this.camera);
            }
        },
        // Gestisce il ridimensionamento della finestra
        handleWindowResize() {
            this.camera.aspect = window.innerWidth / window.innerHeight;
            this.camera.updateProjectionMatrix();
            this.renderer.setSize(window.innerWidth, window.innerHeight);
        },
        // Recupera le radio dalla fonte dati
        async fetchRadios() {
            const response = await fetch('https://de1.api.radio-browser.info/json/stations/search?limit=400&hidebroken=true&has_geo_info=true&order=clickcount&reverse=true');
            const data = await response.json();
            return data;
        },
        // Ottiene le radio e aggiunge i marker alla scena 3D
        async getRadios() {
            try {
                this.radio = await this.fetchRadios();
                this.radio.forEach(radio => {
                    this.addMarker(radio.geo_long, radio.geo_lat, radio);
                    radio.playing = false;
                    radio.audioPlayer = new Audio();
                });
            } catch (error) {
                console.error('Error fetching radios:', error);
            }
        },
        // Gestisce il toggle tra play e pausa per una radio
        togglePlayPause(radio) {
            if (radio.playing) {
                this.pauseRadio(radio);
            } else {
                this.pauseAllRadios(); // Interrompe tutte le altre radio
                this.playRadio(radio);
            }
        },
        // Avvia la riproduzione audio per una radio
        playRadio(radio) {
            // Carica l'URL dell'audio
            const audioUrl = radio.url_resolved || radio.url;
            // Verifica se l'URL è HLS e se il browser lo supporta
            if (audioUrl.includes('m3u8')) {
                if (Hls.isSupported()) {
                    const hls = new Hls();
                    hls.loadSource(audioUrl);
                    hls.attachMedia(radio.audioPlayer);
                } else {
                    console.error('HLS is not supported in this browser.');
                }
            } else {
                // Imposta l'URL dell'audio e avvia la riproduzione
                radio.audioPlayer.src = audioUrl;
            }
            radio.audioPlayer.play()
                .then(() => {
                    console.log('Audio started playing');
                    radio.playing = true;
                })
                .catch(error => {
                    console.error('Error playing audio:', error);
                    if (error.name === 'NotAllowedError') {
                        console.error('Please ensure that the audio playback is allowed in your browser settings.');
                    }
                });
        },
        // Ottiene l'immagine della radio, mostrando una gif se è in riproduzione
        getRadioImage(radio) {
            if (radio.playing) {
                return 'https://whiz-kid.de/images/sound.gif';
            } else {
                return radio.favicon ? radio.favicon : "https://cdn-icons-png.freepik.com/256/508/508206.png?semt=ais_hybrid";
            }
        },
        // Interrompe la riproduzione per una radio
        pauseRadio(radio) {
            radio.audioPlayer.pause();
            radio.playing = false;
        },
        // Interrompe la riproduzione per tutte le radio
        pauseAllRadios() {
            this.radio.forEach(radio => {
                if (radio.playing) {
                    this.pauseRadio(radio);
                }
            });
        },
    },
    // Metodo eseguito al momento della creazione del componente
    created() {
        this.getRadios();
    },
}
</script>

<style scoped>
/* Stili per la barra di navigazione */
.navbar {
    position: fixed;
    bottom: 0; /* Posiziona la navbar in basso */
    left: 0;
    width: 100%;
    background-color: #fff;
    box-shadow: 0 -2px 5px rgba(0, 0, 0, 0.3); /* Sposta l'ombra verso il basso */
    display: flex;
    align-items: center;
    justify-content: space-around;
    padding: 10px;
}

/* Stili per il logo della radio */
.radio-logo {
    width: 50px;
    height: 50px;
    border-radius: 25px;
}
</style>
