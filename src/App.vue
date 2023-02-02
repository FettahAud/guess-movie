<script setup>
import { Configuration, OpenAIApi } from "openai";
import { onMounted, ref } from "vue";
import party from "party-js";
const emoji = ref();
const userInput = ref();
const submit = ref();
const particles = ref();

const answer = ref(false);

const configuration = new Configuration({
  apiKey: import.meta.env.VITE_OPENAI_API_KEY, // your api key ,
});
const openai = new OpenAIApi(configuration);
const movies = [];
const currentMovie = ref("");

const getEmoji = async (movie) => {
  const response = await openai.createCompletion({
    model: "text-davinci-003",
    prompt: `Convert movie titles into emoji.
     ${movie}`,
    temperature: 0.8,
    max_tokens: 60,
    top_p: 1.0,
    frequency_penalty: 0.0,
    presence_penalty: 0.0,
    stop: ["\n"],
  });
  return response.data.choices;
};

const setNewMovie = () => {
  currentMovie.value = movies[Math.round(Math.random() * movies.length - 1)];
};

const checkInput = () => {
  if (
    userInput.value.value.toLowerCase() === currentMovie.value.toLowerCase()
  ) {
    party.confetti(particles.value, {
      gravity: 10,
    });
    nextMovie();
  } else {
    userInput.value.disabled = true;
    submit.value.disabled = true;
    answer.value = true;
  }
};

const nextMovie = () => {
  answer.value = false;
  userInput.value.disabled = false;
  submit.value.disabled = false;
  setNewMovie();
  userInput.value.value = "";
  getEmoji(currentMovie.value).then((response) => {
    emoji.value.textContent = response[0].text;
  });
};

onMounted(() => {
  const options = {
    method: "GET",
    headers: {
      "X-RapidAPI-Key": import.meta.env.VITE_X_RAPID_API_KEY,
      "X-RapidAPI-Host": "imdb-top-100-movies.p.rapidapi.com",
    },
  };

  fetch("https://imdb-top-100-movies.p.rapidapi.com/", options)
    .then((response) => response.json())
    .then((response) => {
      response.map((item) => {
        movies.push(item.title);
      });
      setNewMovie();
      getEmoji(currentMovie.value).then((response) => {
        emoji.value.textContent = response[0].text;
      });
    })
    .catch((err) => console.error(err));
});
</script>

<template>
  <div ref="particles" id="particles"></div>
  <div class="app">
    <h1>Guess the movie name</h1>
    <h2 ref="emoji">Loading...</h2>
    <div class="form">
      <input ref="userInput" type="text" />
      <button @click="checkInput" ref="submit">Submit</button>
      <button @click="nextMovie" class="restart">⏭</button>
    </div>
    <h2 v-show="answer" class="answer">The movie was {{ currentMovie }}</h2>
  </div>
</template>

<style lang="scss" scoped>
#particles {
  width: 100px;
  height: 100px;
  position: absolute;
  top: 42%;
  left: 50%;
  transform: translate(-50%, -50%);
  pointer-events: none;
}
.app {
  width: 100%;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  gap: 10px;
  color: #eeeeee;

  .form {
    display: flex;
    align-items: center;
    gap: 20px;
    input {
      padding: 5px 5px 5px 10px;
      background: #393e46;
      border-radius: 8px;
      color: #eeeeee;
      outline: none;
      border: 0;
    }
    button {
      padding: 8px;
      background: #393e46;
      border: 1px solid #eeeeee;
      outline: none;
      color: #eeeeee;
      border-radius: 8px;
      cursor: pointer;
      &.restart {
        background: none;
        border: 0;
        padding: 0;
        font-size: 24px;
        margin-left: -10px;
      }
    }
    input:disabled,
    button:disabled {
      cursor: not-allowed;
    }
  }
  .answer {
    color: #c3f584;
  }
}
</style>
