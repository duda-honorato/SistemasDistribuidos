//Axios - Handling API Requests * If it were a web application - Fetch
// Readline - User interaction via terminal
const axios = require('axios');
const readline = require('readline');

// API/Address
const OMDB_API_KEY = 'a34b7030';
const TMDB_API_KEY = '95afc5a0481e62bbd8b2923b6adea787';

// Found Data - OMDB
async function MovieFromOMDB(title, year) {
    const url = `http://www.omdbapi.com/?t=${title}&y=${year}&apikey=${OMDB_API_KEY}`;
    const response = await axios.get(url);
    return response.data;
}

// Found Data - TMDB
async function ReviewsFromTMDB(movieId) {
    const url = `https://api.themoviedb.org/3/movie/${movieId}/reviews?api_key=${TMDB_API_KEY}`;
    const response = await axios.get(url);

    console.log('Resposta de reviews do TMDB:', response.data);

    if (response.data.results && response.data.results.length > 0) {
        //3 reviews
        return response.data.results.slice(0, 3).map(review => review.content);
    } else {
        return ['Sem reviews disponíveis'];
    }
}

const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

// Call to user
rl.question('Digite o título do filme: ', (title) => {
    rl.question('Digite o ano do filme: ', (year) => {
        // Chama a função com o título e ano fornecidos
        getMovieInfo(title, parseInt(year)).finally(() => {
            rl.close();
        });
    });
});

async function getMovieInfo(title, year) {
    try {
        const movieData = await MovieFromOMDB(title, year);

        if (movieData.Response === 'False') {
            console.log('Filme não encontrado no OMDB.');
            return;
        } // Exit - Test

        // TMDB Movie ID from Title and Year
        const movieSearch = await axios.get(`https://api.themoviedb.org/3/search/movie?query=${title}&year=${year}&api_key=${TMDB_API_KEY}`);
        const movieId = movieSearch.data.results[0]?.id;

        if (!movieId) {
            console.log('Filme não encontrado no TMDB.');
            return;
        } //Exit - Test

        // Get TMDB reviews
        const reviews = await ReviewsFromTMDB(movieId);

        // The Result
        const { Title, Year, Plot } = movieData;
        const movieInfo = {
            Name: Title,
            Year: parseInt(Year),
            Synopsis: Plot,
            Reviews: reviews
        };

        // Show the result
        console.log(JSON.stringify(movieInfo, null, 2));

    } catch (error) {
        console.error('Erro ao obter dados:', error.message);
    }
}
