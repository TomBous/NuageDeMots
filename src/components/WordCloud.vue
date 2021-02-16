<template>
  <div>
    <div class="row">
      <div class="column center vw45">
        <h1>Générateur de nuages de mots</h1>
        <textarea v-model="text" name="text" id="text"></textarea>
        <v-container class="px-0" fluid>
          <v-switch
            v-model="switch1"
            :label="`Switch 1: ${switch1.toString()}`"
            color="blue"
          ></v-switch>
          <v-slider
            v-model="ex3.val"
            :label="ex3.label"
            :thumb-color="ex3.color"
            thumb-label="always"
            :min="ex3.min"
            :max="ex3.max"
            ticks="always"
            step="1"
          ></v-slider>
        </v-container>
        <Slider orientation='horizontal'/>
        <button @click="generate()">Génerer</button>
      </div>
      <div>
        <div class="container column center">
          <div id="wordcloud"></div>
          <canvas id="canvas" width="600" height="400"></canvas>
        </div>
      </div>
    </div>
  </div>
</template>


<script>
import WordCloud from "wordcloud";


export default {
  data() {
    return {
      text: "tototoot",
      wordcloud: WordCloud,
      switch1: false,
      ex3: {
        label: "color",
        val: 5,
        color: "orange darken-3",
        min: 0,
        max: 30,
      },
    };
  },
  methods: {
    generate() {
      const string = this.text;
      const regexNotWord = /[?():!=*,\s.&'’»«"#[\]_/\\\-;\d}{+]+/gm; // et merge les whites spaces | [^a-zA-z\séèêîûôïëàç] -> l'inverse moins performant
      const regexNomPropres = /([A-Z]{1}[\w]+)(\s)([A-Z]{1}[\w]+)/gm; // cette regex contient 3 groupes (). Le groupe 2 ($2) c'est l'espace entre prenom et nom (ex: Science Po)
      let sanitizedString = string
        .replace(regexNomPropres, "$1~$3")
        .replace(regexNotWord, " ")
        .trim()
        .toLocaleLowerCase();
      const words = sanitizedString.split(" ");
      const longWords = words.filter((word) => word.length > 1);

      //Filtres blacklists
      const motsInutiles = [
        "se",
        "ce",
        "y",
        "en",
        "ceci",
        "cela",
        "ça",
        "tous",
        "tout",
        "tel",
        "tels",
        "que",
        "de",
        "et",
        "plus",
        "sur",
        "au",
        "aux",
        "qui",
        "des",
        "mais",
        "ou",
        "donc",
        "car",
        "donc",
        "aussi",
        "qu",
        "pour",
        "dans",
        "si",
        "avec",
        "par",
        "hui",
        "aujourd",
      ];
      const pronomsPerso = [
        "je",
        "me",
        "moi",
        "toi",
        "tu",
        "ils",
        "il",
        "elle",
        "elles",
        "vous",
        "nous",
        "leur",
        "eux",
        "on",
      ];
      const verbesAxiliaire = [
        "ai",
        "as",
        "avons",
        "avez",
        "ont",
        "suis",
        "es",
        "est",
        "sommes",
        "êtes",
        "sont",
      ];
      /* const pronomsPossessif = [
        "mien",
        "tien",
        "siens",
        "toi",
        "miens",
        "tiens",
        "notre",
        "mienne",
        "tienne",
        "vôtre",
        "nôtre",
        "votre",
      ]; */
      const determinants = [
        "le",
        "la",
        "les",
        "un",
        "une",
        "du",
        "ce",
        "cet",
        "cette",
        "ces",
        "quel",
        "quels",
        "quelle",
        "quelles",
      ];

      const importantWords = longWords
        .filter((word) => !motsInutiles.includes(word))
        .filter((word) => !pronomsPerso.includes(word))
        .filter((word) => !verbesAxiliaire.includes(word))
        .filter((word) => !determinants.includes(word)); // N'est pas dans mot motsInutiles
      console.log(importantWords);
      const list = this.createList(importantWords, 40);
      let convertedList = this.listConverter(list, 10);
      console.log("Occurences : ", list);
      console.log("Converted : ", convertedList);
      const options = {
        list: convertedList,
        fontFamily: "Caveat",
        //fontWeight: 400,
        gridSize: 10,
        minSize: 1,
        weightFactor: 2,
        backgroundColor: "white",
        minRotation: -0.2,
        maxRotation: 0.2,
        rotateRatio: 0.8,
        drawOutOfBound: true,
        shape: "cardioid",
        color: function () {
          return ["#d50000", "#002171", "#0d47a1"][
            Math.floor(Math.random() * 3)
          ];
        },
      };
      this.wordcloud(document.getElementById("wordcloud"), options);
      setTimeout(() => {
        this.wordcloud(document.getElementById("canvas"), options);
      }, 1000);
    },
    createList(wordsArray, limitNumber) {
      let list = [];
      // Populate array
      wordsArray.forEach((word) => {
        const indexOfItem = this.isInList(list, word);
        if (indexOfItem < 0) {
          list.push([word, 1]);
        } else {
          list[indexOfItem][1] += 1;
        }
      });
      // Du plus d'occurrences au moins
      list.sort((a, b) => b[1] - a[1]);
      // Gestion des pluriels
      list.splice(limitNumber * 5); // On limite la taille de l'array pour les perfs
      list.forEach((item, currentIndex) => {
        if (item[0].charAt(item[0].length - 1) == "s") {
          // si le mot termine par "s"
          const indexOfSingular = this.isInList(
            list,
            item[0].substr(0, item[0].length - 1)
          ); // On trouve la position du singulier
          if (indexOfSingular >= 0) {
            list[indexOfSingular][1] += item[1]; // On ajoute le nb occurrences au mot singulier
            list.splice(currentIndex, 1); // on supprime le pluriel
          }
        }
      });
      list.splice(limitNumber); // On reduit la liste à la taille voulue
      // On remet l'espace dans les noms propres
      list.forEach((item) => {
        item[0] = item[0].replace("~", " ");
      });
      return list;
    },
    listConverter(list, fontSizeTarget) {
      const minOccurence = list[list.length - 1][1];
      const minTempOutput =
        Math.log(minOccurence + 1) * Math.log(minOccurence + 1);
      const convertedList = JSON.parse(JSON.stringify(list)); // Le spread opérator [...list] ne marche qu'avec un array simple
      const divider = minTempOutput / fontSizeTarget;
      convertedList.forEach((item) => {
        //item[1] = item[1] / divider
        console.log(Math.log(item[1]));
        item[1] = (Math.log(item[1]) * Math.log(item[1])) / divider;
      });
      return convertedList;
    },
    isInList(list, word) {
      return list.findIndex((item) => item[0] == word);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Caveat:wght@600&display=swap");
* {
  box-sizing: border-box;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
textarea {
  width: 40vw;
  height: 20vh;
  border-radius: 10px;
  border: 2px solid #d9d9d9;
  padding: 0.5em;
}
canvas {
  width: 600px;
  height: 400px;
}
button {
  margin: 20px 10px;
}
#wordcloud {
  width: 600px;
  height: 400px;
}
.row {
  display: flex;
  flex-flow: row wrap;
}
.column {
  display: flex;
  flex-flow: column wrap;
}
.center {
  align-items: center;
}
.container {
  width: 50vw;
}
.vw45 {
  width: 45vw;
}
</style>