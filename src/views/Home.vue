<template>
    <div id="blur"></div>
    <div id="bannerSearch">
        <input type="text" id="search" @keyup.enter="search" ref="searchInput" value="21 pilots">
        <button @click="search">Search</button>
    </div>
    <div id="searchResult"></div>
    <YoutubePlayer ref="youtubePlayer" id="player"/>
</template>

<script>
import YoutubePlayer from '../components/YoutubePlayer.vue';

export default ({
    name: "MusicPage",
    components: {
        YoutubePlayer
    },
    data() {
        return {
            current_video_id: "",
            video_playing: false,
            API_KEY: 'AIzaSyA8apjRRfjCHmu6M_4q_r3kUbnO_qJ7xfk',
            API_URL: 'https://www.googleapis.com/youtube/v3/search',
            VIDEO_DETAILS_URL: 'https://www.googleapis.com/youtube/v3/videos',
            serpAPI_KEY: '777faa1853ab2cc4bfb9c2b265b2147cc6167016356d283bbfc7cb61903009a8'
        };
    },
    methods: {
        init(){
            document.getElementById("blur").onclick = () => {
                if (this.video_playing) {
                    this.video_playing = false
                    document.getElementById("player").style.display = "none"
                    document.getElementById("blur").style.display = "none"
                }
            }
        },
        async getRecommendedVideo() {
            const url = `https://www.googleapis.com/youtube/v3/search?part=snippet&relatedToVideoId=${this.current_video_id}&type=video&key=${this.API_KEY}`;
            try {
                const response = await fetch(url);
                const data = await response.json();
                if (data.items && data.items.length > 0) {
                    return data.items[0].id.videoId; // Retourne l'ID de la première vidéo recommandée
                }
            } catch (error) {
                console.error("Erreur lors de la récupération des recommandations :", error);
            }
            return null;
        },
        async peopleAlsoSearchFor(){
            const url = `https://serpapi.com/search.json?engine=youtube&search_query=${this.$refs.searchInput.value}&api_key=${this.serpAPI_KEY}`;
            try {
                const response = await fetch(url);
                console.log(response)
            } catch (error) {
                console.error("Erreur lors de la récupération des recommandations :", error);
            }
            return null;
        },
        async APIytSearch(input) {
            // Construire l'URL avec les paramètres
            const url = `${this.API_URL}?part=snippet&type=video&q=${encodeURIComponent(input)}&key=${this.API_KEY}&maxResults=100`;
            
            // Effectuer une requête fetch
            const response = await fetch(url);
            if (!response.ok) {
                throw new Error(`HTTP error! Status: ${response.status}`);
            }
            
            const searchData = await response.json();

            // Récupérer les IDs des vidéos trouvées
            const videoIds = searchData.items.map(item => item.id.videoId).join(',');

            // Étape 2 : Obtenir les détails des vidéos pour vérifier la catégorie
            const videoDetailsResponse = await fetch(
                `${this.VIDEO_DETAILS_URL}?part=snippet&id=${videoIds}&key=${this.API_KEY}`
            );
            const videoDetailsData = await videoDetailsResponse.json();

            // Filtrer uniquement les vidéos ayant la catégorie musique (ID: 10)
            return videoDetailsData.items.filter(video => video.snippet.categoryId === '10');
        },
        async search() {
            try {
                let musicVideos = await this.APIytSearch(this.$refs.searchInput.value)

                // Afficher les résultats
                console.log('Résultats de la recherche :', musicVideos);

                this.$refs.youtubePlayer.setPlayList(musicVideos);
                
                document.getElementById("searchResult").innerHTML = ""

                // Exemple d'affichage de titres et de vidéos
                musicVideos.forEach(item => {
                    console.log(`Titre : ${item.snippet.title}`);
                    console.log(`Lien : https://www.youtube.com/watch?v=${item.id.videoId}`);

                    let div = document.createElement("div")
                    div.id = item.id
                    div.className = "searchOneResult"

                    div.onclick = () => {
                        this.video_playing = true
                        this.$refs.youtubePlayer.playNewVideo(item.id);
                        document.getElementsByTagName("body")[0].style.overflow = "hidden"
                        document.getElementById("player").style.display = "block"
                        document.getElementById("blur").style.display = "block"
                    }

                    let img = document.createElement("img")
                    img.src = item.snippet.thumbnails.default.url
                    div.appendChild(img)

                    let divDesc = document.createElement("div")
                    divDesc.className = "divDesc"

                    let h2 = document.createElement("h2")
                    h2.textContent = item.snippet.title

                    divDesc.appendChild(h2)

                    div.appendChild(divDesc)

                    document.getElementById("searchResult").appendChild(div)
                    
                });
            } catch (error) {
                console.error('Erreur lors de la recherche YouTube :', error);
            }
        }
    },
    async mounted() {
        this.init()
    }
});
</script>

<style scopped>
#search {
    border: 1px solid black;
}

#bannerSearch {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #EEEEEE;
    height: 60px;
    position: sticky;
    left: 0;
    right: 0;
    top: 0;
}

#searchResult {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-around;
}

.searchOneResult {
    display: flex;
    flex-direction: row;
    align-items: center;
    width: 50%;
    margin-top: 10px;
    margin-bottom: 10px;
    border: 1px solid black
}

.searchOneResult h2{
    margin-left: 10px;
}

#player {
    display: none;
    position: fixed;
    top: 100px;
    bottom: 100px;
    left: 100px;
    right: 100px; 
    width: calc(100% - 200px); 
    height: calc(100% - 200px);
    z-index: 3;
}

#blur{
    width: 100vw;
    height: 100vh;
    background-color: black;
    position: absolute;
    z-index: 2;
    opacity: 0.5;
    filter: blur(100px);
    display: none;
}
</style>
  