<template>
  <v-container fluid>
    <v-row style="large">
      <v-col cols="4" sm="4">
        <v-textarea
          solo
          name="text"
          v-model="text"
          placeholder="Votre texte ici"
        ></v-textarea>
        <h2>Mots à retirer</h2>
        <v-row>
          <v-col>
            <v-switch
              v-model="filterPronoms"
              :label="'Pronoms personnels'"
              color="cyan darken-1"
            ></v-switch>
            <v-switch
              v-model="filterPossessifs"
              :label="'Pronoms possessifs'"
              color="cyan darken-1"
            ></v-switch>
          </v-col>
          <v-col>
            <v-switch
              v-model="filterVerbAux"
              :label="'Verbes auxiliaires'"
              color="cyan darken-1"
            ></v-switch>
            <v-switch
              v-model="filterDeterminants"
              :label="'Déterminants'"
              color="cyan darken-1"
            ></v-switch>
          </v-col>
        </v-row>
        <h2>Configuration</h2>
        <v-slider
          v-model="zoom.val"
          :label="zoom.label"
          :thumb-color="zoom.color"
          thumb-label="always"
          :min="zoom.min"
          :max="zoom.max"
          ticks="always"
          step="1"
        ></v-slider>
        <v-slider
          v-model="nbWord.val"
          :label="nbWord.label"
          :thumb-color="nbWord.color"
          thumb-label="always"
          :min="nbWord.min"
          :max="nbWord.max"
          ticks="always"
          step="5"
        ></v-slider>
        <v-row>
        <!-- Color picker 1 -->
        <v-text-field
          v-model="color1.color"
          v-mask="hexMask"
          hide-details
          class="input-color"
          solo
        >
          <template v-slot:append>
            <v-menu
              v-model="color1.menu"
              top
              nudge-bottom="105"
              nudge-left="16"
              :close-on-content-click="false"
            >
              <template v-slot:activator="{ on }">
                <div :style="swatchStyle1" v-on="on" />
              </template>
              <v-card>
                <v-card-text class="pa-0">
                  <v-color-picker id="color1" v-model="color1.color" flat />
                </v-card-text>
              </v-card>
            </v-menu>
          </template>
        </v-text-field>
        <!-- Color picker 2 -->
        <v-text-field
          v-model="color2.color"
          v-mask="hexMask"
          hide-details
          class="input-color"
          solo
        >
          <template v-slot:append>
            <v-menu
              v-model="color2.menu"
              top
              nudge-bottom="105"
              nudge-left="16"
              :close-on-content-click="false"
            >
              <template v-slot:activator="{ on }">
                <div :style="swatchStyle2" v-on="on" />
              </template>
              <v-card>
                <v-card-text class="pa-0">
                  <v-color-picker v-model="color2.color" flat />
                </v-card-text>
              </v-card>
            </v-menu>
          </template>
        </v-text-field>
        <!-- Color picker 3 -->
        <v-text-field
          v-model="color3.color"
          v-mask="hexMask"
          hide-details
          class="input-color"
          solo
        >
          <template v-slot:append>
            <v-menu
              v-model="color3.menu"
              top
              nudge-bottom="105"
              nudge-left="16"
              :close-on-content-click="false"
            >
              <template v-slot:activator="{ on }">
                <div :style="swatchStyle3" v-on="on" />
              </template>
              <v-card>
                <v-card-text class="pa-0">
                  <v-color-picker v-model="color3.color" flat />
                </v-card-text>
              </v-card>
            </v-menu>
          </template>
        </v-text-field>
        </v-row>

        <v-btn @click="generate()" color="cyan darken-1" elevation="2" dark
          >Génerer</v-btn
        >
        <v-btn
          @click="downloadCanvas()"
          color="cyan darken-1"
          elevation="2"
          dark
          >Enregistrer</v-btn
        >
      </v-col>

      <v-col cols="8" sm="8">
        <div class="container column center">
          <div id="wordcloud"></div>
          <canvas id="canvas" width="1024" height="600"></canvas>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>


<script>
import WordCloud from "wordcloud";

export default {
  data() {
    return {
      text: "",
      wordcloud: WordCloud,
      filterPronoms: true,
      filterVerbAux: true,
      filterDeterminants: true,
      filterPossessifs: true,
      zoom: {
        label: "Zoom",
        val: 3,
        color: "cyan darken-1",
        min: 0,
        max: 10,
      },
      nbWord: {
        label: "Nombre de mots",
        val: 40,
        color: "cyan darken-1",
        min: 10,
        max: 60,
      },
      hexMask: "!#XXXXXXXX",
      color1: {
        color: "#002171FF",
        menu: false,
      },
      color2: {
        color: "#d50000FF",
        menu: false,
      },
      color3: {
        color: "#0d47a1FF",
        menu: false,
      },
    };
  },
  mounted() {},
  computed: {
    swatchStyle1() {
      const { color1 } = this;
      return {
        backgroundColor: color1.color,
        cursor: "pointer",
        height: "30px",
        width: "30px",
        borderRadius: color1.menu ? "50%" : "4px",
        transition: "border-radius 200ms ease-in-out",
      };
    },
    swatchStyle2() {
      const { color2 } = this;
      return {
        backgroundColor: color2.color,
        cursor: "pointer",
        height: "30px",
        width: "30px",
        borderRadius: color2.menu ? "50%" : "4px",
        transition: "border-radius 200ms ease-in-out",
      };
    },
    swatchStyle3() {
      const { color3 } = this;
      return {
        backgroundColor: color3.color,
        cursor: "pointer",
        height: "30px",
        width: "30px",
        borderRadius: color3.menu ? "50%" : "4px",
        transition: "border-radius 200ms ease-in-out",
      };
    },
  },
  methods: {
    generate() {
      const state = this
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
      const pronomsPossessif = [
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
      ];
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
      let importantWords = longWords.filter(
        (word) => !motsInutiles.includes(word)
      );
      if (this.filterPronoms) {
        importantWords = importantWords.filter(
          (word) => !pronomsPerso.includes(word)
        );
      }
      if (this.filterVerbAux) {
        importantWords = importantWords.filter(
          (word) => !verbesAxiliaire.includes(word)
        );
      }
      if (this.filterDeterminants) {
        importantWords = importantWords.filter(
          (word) => !determinants.includes(word)
        ); // N'est pas dans mot motsInutiles
      }
      if (this.filterPossessifs) {
        importantWords = importantWords.filter(
          (word) => !pronomsPossessif.includes(word)
        );
      }
      console.log(importantWords);
      const list = this.createList(importantWords, this.nbWord.val);
      let convertedList = this.listConverter(list, 10);
      console.log("Occurences : ", list);
      console.log("Converted : ", convertedList);
      const options = {
        list: convertedList,
        fontFamily: "Caveat",
        //fontWeight: 400,
        gridSize: Math.round((16 * 700) / 1024),
        minSize: 1,
        weightFactor: this.zoom.val,
        origin: [1024 / 2, 600 / 4.5],
        backgroundColor: "white",
        minRotation: -0.2,
        maxRotation: 0.2,
        rotateRatio: 0.8,
        drawOutOfBound: true,
        shape: "cardioid",
        color: function () {
          return [state.color1.color, state.color2.color, state.color3.color][
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
      const FutureMinOutput =
        Math.log(minOccurence + 1) * Math.log(minOccurence + 1);
      const convertedList = JSON.parse(JSON.stringify(list)); // Le spread opérator [...list] ne marche qu'avec un array simple
      const divider = FutureMinOutput / fontSizeTarget; // On fixe la taille de l'occurence min
      convertedList.forEach((item) => {
        item[1] = (Math.log(item[1]) * Math.log(item[1])) / divider;
      });
      return convertedList;
    },
    isInList(list, word) {
      return list.findIndex((item) => item[0] == word);
    },
    downloadCanvas() {
      var canvas = document.getElementById("canvas");
      canvas.toBlob((blob) => {
        const downloadUrl = URL.createObjectURL(blob);
        this.download("image.png", downloadUrl);
      });
    },
    download(filename, downloadUrl) {
      var element = document.createElement("a");
      element.setAttribute("href", downloadUrl);
      element.setAttribute("download", filename);
      element.style.display = "none";
      document.body.appendChild(element);
      element.click();
      document.body.removeChild(element);
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

canvas {
  display: block;
  position: relative;
  overflow: hidden;
  margin-right: auto;
  margin-left: auto;
}
button {
  margin: 20px 10px;
}
#wordcloud {
  position: relative;
  width: 1024px;
  margin-left: auto;
  margin-right: auto;
  height: 600px;
}
.large {
  width: 90vw;
}
.flex {
  display: flex;
  flex-flow: row column;
  align-items: center;
}
.margin-slider {
  margin-right: 10%;
}
.input-color {
  width: 33%;
}
</style>