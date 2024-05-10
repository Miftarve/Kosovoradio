<!-- Questo template rappresenta l'interfaccia grafica dell'applicazione -->
<template>
    <!-- Il container della scena 3D renderizzata con Three.js -->
    <div ref="container"></div>

    <!-- La barra di navigazione che mostra le informazioni sulla radio selezionata -->
    <div class="navbar" v-if="selectedRadio">
        <!-- Il logo della radio -->
        <img :src="selectedRadio.favicon || defaultImage" class="radio-logo" alt="Radio logo">

        <!-- Il nome della radio -->
        <h4>{{ selectedRadio.name }}</h4>

        <!-- Il pulsante per avviare o mettere in pausa la riproduzione della radio -->
        <button @click="togglePlayPause(selectedRadio)">{{ selectedRadio.playing ? 'Pause' : 'Play' }}</button>

        <!-- L'animazione delle onde sonore che indica che la radio è in riproduzione -->
        <div v-if="selectedRadio.playing" class="sound-wave">
            <div class="bar"></div>
            <div class="bar"></div>
            <div class="bar"></div>
            <div class="bar"></div>
        </div>

        <!-- Il pulsante per aggiungere o rimuovere la radio dai preferiti -->
        <div @click="toggleFavorite(selectedRadio)" class="heart-container">
            <div :class="['heart', { 'liked': selectedRadio.favorite }]">
            </div>
        </div>
    </div>
</template>

<script>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import earthTexture from '../assets/8k_earth_daymap.jpg';
import Hls from 'hls.js';
import defaultImage from '../assets/radio.png';

export default {
    name: 'ThreeJsScene',
    data() {
        return {
            // Inizializzazione delle variabili per la scena 3D
            camera: null,
            renderer: null,
            controls: null,
            earthRadius: 1,
            radios: [],
            selectedRadio: null,
            defaultImage,
        };
    },
    mounted() {
        // Inizializzazione della scena 3D
        this.init();
        this.animate();

        // Recupero delle radio disponibili
        this.getRadios();

        // Aggiunta degli eventi per il ridimensionamento della finestra e il click del mouse
        window.addEventListener('resize', this.handleWindowResize);
        this.raycaster = new THREE.Raycaster();
        this.mouse = new THREE.Vector2();
        window.addEventListener('click', this.onDocumentMouseClick);
    },
    beforeUnmount() {
        // Rimozione degli eventi prima che il componente venga distrutto
        window.removeEventListener('resize', this.handleWindowResize);
        window.removeEventListener('click', this.onDocumentMouseClick);
    },
    methods: {
        // Inizializzazione della scena 3D con Three.js
        init() {
            // Inizializzazione della telecamera, del renderer e dei controlli
            this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            this.camera.position.z = 5;
            this.renderer = new THREE.WebGLRenderer();
            this.renderer.setSize(window.innerWidth, window.innerHeight);
            this.$refs.container.appendChild(this.renderer.domElement);
            this.controls = new OrbitControls(this.camera, this.renderer.domElement);
            this.controls.enableDamping = true;

            // Creazione della sfera terrestre con la mappa
            const scene = new THREE.Scene();
            const geometry = new THREE.SphereGeometry(this.earthRadius, 64, 64);
            const texture = new THREE.TextureLoader().load(earthTexture);
            const material = new THREE.MeshPhongMaterial({ map: texture });
            const earth = new THREE.Mesh(geometry, material);
            scene.add(earth);

            // Aggiunta delle luci alla scena
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
            scene.add(ambientLight);
            const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
            directionalLight.position.set(1, 1, 1).normalize();
            scene.add(directionalLight);

            // Salvataggio della scena creata
            this.scene = scene;
        },

        // Funzione per gestire il click del mouse sulla scena 3D
        onDocumentMouseClick(event) {
            event.preventDefault();
            this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            this.mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
            this.raycaster.setFromCamera(this.mouse, this.camera);
            const intersects = this.raycaster.intersectObjects(this.scene.children);

            if (intersects.length > 0) {
                console.log('Clicked on:', intersects[0].object.userData);
                this.handleMarkerClick(intersects[0]);
            }
        },

        // Funzione per gestire il click sui marker nella scena 3D
        handleMarkerClick(intersectedObject) {
            if (intersectedObject.object.onClick) {
                intersectedObject.object.onClick();
                const previousRadio = this.selectedRadio;
                this.selectedRadio = intersectedObject.object.userData;

                // Reimpostazione del colore del marker precedentemente selezionato a rosso
                if (previousRadio && previousRadio.marker) {
                    previousRadio.marker.material.color.set(0xFF0000); // Rosso
                }

                // Cambio del colore del marker appena cliccato a giallo
                intersectedObject.object.material.color.set(0xFFFF00); // Giallo

                // Controllo se la radio selezionata ha un'URL audio e avvio il caricamento lazy dell'audio
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

        // Funzione per il caricamento lazy dell'audio della radio selezionata
        lazyLoadAudio(radio) {
            console.log('Lazy loading audio for:', radio);
            if (!radio.audioPlayer) {
                console.log('Creating new audio player');
                radio.audioPlayer = new Audio(radio.url_resolved || radio.url);
                console.log('Audio player:', radio.audioPlayer);
                radio.audioPlayer.onloadeddata = () => {
                    console.log('Audio loaded');
                };
                radio.audioPlayer.onerror = () => {
                    console.error('Error loading audio');
                };
            }
            else {
                console.log('Audio player already exists');
            }
        },
        // Funzione per aggiungere un marker sulla sfera terrestre
        addMarker(longitude, latitude, radio) {
            // Verifica se la longitudine e la latitudine non sono null
            if (longitude !== null && latitude !== null) {
                // Calcolo delle coordinate sferiche
                const phi = (90 - latitude) * (Math.PI / 180);
                const theta = (longitude + 180) * (Math.PI / 180);
                const x = -this.earthRadius * Math.sin(phi) * Math.cos(theta);
                const y = this.earthRadius * Math.cos(phi);
                const z = this.earthRadius * Math.sin(phi) * Math.sin(theta);

                // Creazione del marker visibile
                const geometry = new THREE.SphereGeometry(0.01, 32, 32);
                const material = new THREE.MeshBasicMaterial({ color: 0xFF0000 }); // Rosso di default
                const marker = new THREE.Mesh(geometry, material);
                marker.position.set(x, y, z);
                marker.userData = radio;
                marker.onClick = () => {
                    console.log('Clicked on marker:', marker.userData);
                };
                this.scene.add(marker);

                // Memorizzazione del marker nell'oggetto radio
                radio.marker = marker;

                // Creazione di una sfera invisibile per il click
                const invisibleSphereGeometry = new THREE.SphereGeometry(0.4, 32, 32);
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

        // Funzione per l'animazione della scena 3D
        animate() {
            requestAnimationFrame(this.animate);

            // Aggiornamento dei controlli e rendering della scena
            if (this.controls) {
                this.controls.update();
            }

            if (this.renderer && this.scene && this.camera) {
                this.renderer.render(this.scene, this.camera);
            }
        },

        // Funzione per gestire il ridimensionamento della finestra
        handleWindowResize() {
            this.camera.aspect = window.innerWidth / window.innerHeight;
            this.camera.updateProjectionMatrix();
            this.renderer.setSize(window.innerWidth, window.innerHeight);
        },

        // Funzione per ottenere i dati delle radio
        async fetchRadios() {
            const response = await fetch('https://de1.api.radio-browser.info/json/stations/search?limit=700&hidebroken=true&has_geo_info=true&order=clickcount&reverse=true');
            const data = await response.json();
            return data;
        },

        // Funzione per ottenere le radio e aggiungere i marker sulla mappa
        async getRadios() {
            try {
                this.radios = await this.fetchRadios();
                this.radios.forEach(radio => {
                    this.addMarker(radio.geo_long, radio.geo_lat, radio);
                    radio.playing = false;
                    radio.audioPlayer = new Audio();
                });
                this.retrieveFavorites();
            } catch (error) {
                console.error('Error fetching radios:', error);
            }
        },
        // Funzione per avviare o mettere in pausa la riproduzione della radio
        togglePlayPause(radio) {
            if (radio.playing) {
                this.pauseRadio(radio);
            } else {
                this.pauseAllRadios(); // Metti in pausa tutte le altre radio
                this.playRadio(radio);
            }
        },

        // Funzione per avviare la riproduzione della radio
        playRadio(radio) {
            console.log('playRadio called', radio);
            const audioUrl = radio.url_resolved || radio.url;
            console.log('Audio URL:', audioUrl);
            if (audioUrl.includes('m3u8')) {
                if (Hls.isSupported()) {
                    console.log('HLS is supported');
                    const hls = new Hls();
                    hls.loadSource(audioUrl);
                    hls.attachMedia(radio.audioPlayer);
                } else {
                    console.error('HLS is not supported in this browser.');
                }
            } else {
                console.log('Setting audio source:', audioUrl);
                radio.audioPlayer.src = audioUrl;
            }
            // Avvio della riproduzione dell'audio
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

        // Funzione per mettere in pausa la riproduzione della radio
        pauseRadio(radio) {
            radio.audioPlayer.pause();
            radio.playing = false;
        },

        // Funzione per mettere in pausa tutte le radio
        pauseAllRadios() {
            this.radios.forEach(radio => {
                if (radio.playing) {
                    this.pauseRadio(radio);
                }
            });
        },

        // Funzione per aggiungere o rimuovere la radio dai preferiti
        toggleFavorite(radio) {
            radio.favorite = !radio.favorite;
            if (!radio.favorite) {
                this.removeFavorite(radio);
            } else {
                this.saveFavorites();
            }
        },

        // Funzione per salvare i preferiti nell'archiviazione locale
        saveFavorites() {
            const favorites = this.radios.map(radio => ({ changeuuid: radio.changeuuid, favorite: radio.favorite }));
            localStorage.setItem('favorites', JSON.stringify(favorites));
        },

        // Funzione per rimuovere una radio dai preferiti
        removeFavorite(radio) {
            let favorites = JSON.parse(localStorage.getItem('favorites')) || [];
            favorites = favorites.filter(fav => fav.changeuuid !== radio.changeuuid);
            localStorage.setItem('favorites', JSON.stringify(favorites));
        },

        // Funzione per recuperare i preferiti dall'archiviazione locale
        retrieveFavorites() {
            const favorites = JSON.parse(localStorage.getItem('favorites')) || [];
            this.radios.forEach(radio => {
                const fav = favorites.find(f => f.changeuuid === radio.changeuuid);
                radio.favorite = fav ? fav.favorite : false;
            });
        },

        // Funzione chiamata quando il componente viene creato
        created() {
            this.getRadios();
            this.retrieveFavorites();
        }
    }
};
</script>

<style scoped>
.navbar {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    background-color: #fff;
    box-shadow: 0 -2px 5px rgba(0, 0, 0, 0.3);
    display: flex;
    align-items: center;
    justify-content: space-around;
    padding: 10px;
}

.radio-logo {
    width: 50px;
    height: 50px;
    border-radius: 25px;
}

.sound-wave {
    display: flex;
    align-items: center;
    height: 20px;
    margin-left: 10px;
    margin-top: 2px;
}

.bar {
    width: 4px;
    height: 100%;
    margin: 0 2px;
    background-color: #333;
    animation: pulse 0.8s infinite ease-in-out alternate;
}

.bar:nth-child(1) {
    animation-delay: 0s;
}

.bar:nth-child(2) {
    animation-delay: 0.1s;
}

.bar:nth-child(3) {
    animation-delay: 0.2s;
}

.bar:nth-child(4) {
    animation-delay: 0.3s;
}

@keyframes pulse {
    0% {
        transform: scaleY(1);
    }

    100% {
        transform: scaleY(1.5);
    }
}

.heart-container {
    display: inline-block;
    cursor: pointer;
    margin-left: 5px;
    margin-top: 2px;
}

.heart {
    width: 20px;
    height: 18px;
    background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z" fill="%23C1C1C1"/></svg>') center no-repeat;
    background-size: 100%;
}

.heart.liked {
    background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M16.5 3c-1.74 0-3.41.81-4.5 2.09C10.91 3.81 9.24 3 7.5 3 4.42 3 2 5.42 2 8.5c0 3.78 3.4 6.86 8.55 11.54L12 21.35l1.45-1.32C18.6 15.36 22 12.28 22 8.5 22 5.42 19.58 3 16.5 3z" fill="%23FF0000"/></svg>');
}
</style>