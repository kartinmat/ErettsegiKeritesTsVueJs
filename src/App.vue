<template>
  <div id="app" style="background-color:#add8e6;">
    <b-navbar toggleable="lg" type="dark" variant="primary">
      <b-navbar-brand href="#">Kerítés feladat</b-navbar-brand>
      <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>
      <b-collapse id="nav-collapse" is-nav>
        <b-navbar-nav>
          <b-nav-item href="kerites.txt">kerites.txt</b-nav-item>
          <b-nav-item href="Kerítés_fel.pdf">Feladat.pdf</b-nav-item>
          <b-nav-item href="Kerítés_jav.pdf">Javítási.pdf</b-nav-item>
          <b-nav-item href="https://github.com/nitslaszlo/ErettsegiKeritesTsVueJs">GitHub</b-nav-item>
          <b-nav-item href="https://github.com/nitslaszlo/JedlikVueJsStarter">SDK</b-nav-item>
        </b-navbar-nav>
      </b-collapse>
    </b-navbar>
    <TxtReader label="Forrás (kerites.txt):" placeholder="Bedobó mező" @load="txtSorai = $event" />
    <div v-if="mutat" id="megoldas" class="h-100 row align-items-center" style="margin: auto;">
      <b-carousel
        class="h-100 row align-items-center"
        id="tartalom"
        v-model="slide"
        controls
        indicators
        background="#007176"
        img-width="800"
        img-height="300"
        style="text-shadow: 1px 1px 2px #333; width: 800px; height: 300px;"
      >
        <b-carousel-slide img-blank>
          <p>
            2. feladat
            <br />
            Az eladott telkek száma: {{ telkek.length }}
          </p>
        </b-carousel-slide>
        <b-carousel-slide img-blank>
          <p>
            3. feladat
            <br />
            A {{ utolsoTelek.oldal }} oldalon adták el az utolsó telket
            <br />
            Az utolsó telek házszáma: {{ utolsoTelek.hazSzama }}
          </p>
        </b-carousel-slide>
        <b-carousel-slide img-blank>
          <p>
            4. feladat
            <br />
            A szomszédossal egyezik a kerítés színe: {{ ugyanOlyanSzinuKerites }}
          </p>
        </b-carousel-slide>
        <b-carousel-slide img-blank>
          <p>
            5. feladat
            <br />Adjon meg egy házszámot!
            <input
              v-model="hazszamInputStr"
              type="number"
              min="1"
              max="117"
            />
            <br />
            A kerírés színe / állapota: {{ keritesSzineAllapota }}
            <br />
            Egy lehetséges festési szín: {{ lehetsegesFestesiSzin }}
          </p>
        </b-carousel-slide>
      </b-carousel>
    </div>

    <p v-if="mutat" class="h-100 row align-items-center">
      <TxtWriter
        style="margin: auto; margin-top: 50px; background-color: #007176; color: white; border: #007176;"
        title="utcakep.txt állomány mentése"
        :content="utcakep"
        filename="utcakep.txt"
      />
    </p>

    <!-- Megoldás DIV -->

    <!-- Nem a feladat része : -->
    <div v-if="mutat" id="egyebek">
      <pre>
        <div class="h-100 row align-items-center" style="text-align: center;">
          <b-button v-b-toggle.collapse-1 variant="primary" style="background-color: #007176; color: white; margin-left: 28%;">utcakep.txt fájl</b-button>
          <b-button v-b-toggle.collapse-2 variant="primary" style="background-color: #007176; color: white; margin-left: 20%;">kerites.txt fájl</b-button>
        </div>
        <div>
          <b-collapse id="collapse-1" class="mt-2">
            <b-card style="background-color: #007176; color: white;">
              <p class="h-100 row" style="background-color: #007176; color: white;">
                {{ utcakep }}
              </p>
            </b-card>
          </b-collapse>

          
          <b-collapse id="collapse-2" class="mt-2">
            <b-card style="background-color: #007176; color: white;">
              <p class="h-100 row" style="background-color: #007176; color: white;">
                {{ txtSorai }}
              </p>
            </b-card>
          </b-collapse>
        </div>

        

        
      </pre>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";
import Telek from "./telek";
import TxtReader from "./components/TxtReader.vue";
import TxtWriter from "./components/TxtWriter.vue";

// eslint-disable-next-line
@Component({ components: { TxtReader, TxtWriter } })
export default class App extends Vue {
  private telkek: Telek[] = [];
  private txtSorai: string = ""; // Watch végett nem lehet ékezetes azonosító! (pl.: forrás)!
  private mutat: boolean = false;
  // 5. feladat:
  private hazszamInputStr: string = "83";
  // 6. feladat:
  private utcakep: string = "";

  @Watch("txtSorai", { immediate: true, deep: true })
  private haForrasValtozik(val: string) {
    if (val !== "") this.feldolgoz();
  }

  private feldolgoz(): void {
    try {
      Telek.hazszamReset(); // statikus mezők visszaállítása
      this.txtSorai.split("\n").forEach(i => {
        const aktSor: string = i.trim();
        if (aktSor.length > 0) this.telkek.push(new Telek(aktSor));
      });
      // 6. utcakep generalasa
      let sor1: string = "";
      let sor2: string = "";
      for (const i of this.telkek.filter(x => x.paratlanOldali)) {
        sor1 += "".padEnd(i.telekSzelessege, i.keritesSzine);
        sor2 += i.hazSzama.toString().padEnd(i.telekSzelessege, " ");
      }
      this.utcakep = `${sor1}\n${sor2}\n`;

      this.mutat = true;
    } catch (error) {
      this.mutat = false;
      this.telkek = [];
      this.txtSorai = "";
      window.alert("Hibás forrás!");
      location.reload();
    }
  }

  private get utolsoTelek(): Telek {
    return this.telkek[this.telkek.length - 1];
  }

  private get ugyanOlyanSzinuKerites(): number {
    let elozoTelek: Telek; // induláskor undefined értékű
    // elozoKerites! -> igaz, ha értéke nem null vagy undefined
    for (const aktTelek of this.telkek.filter(x => x.paratlanOldali)) {
      if (
        elozoTelek! &&
        aktTelek.keritesSzine !== "#" &&
        aktTelek.keritesSzine !== ":" &&
        aktTelek.keritesSzine === elozoTelek!.keritesSzine
      ) {
        return elozoTelek!.hazSzama;
      } else elozoTelek = aktTelek; // ha még undefined az ElozoTelek
    }
    return -1; // id a feladatkiírás szerint már nem juthat el
  }

  private get keritesSzineAllapota(): string {
    const hazszamInput: number = parseInt(this.hazszamInputStr, 10);
    const keresettTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput);
    if (keresettTelek.length !== 0) {
      return keresettTelek[0].keritesSzine;
    } else {
      return "Nincs ilyen házszám!"; // ez nem volt feladat, de így szép
    }
  }

  private get lehetsegesFestesiSzin(): string {
    const hazszamInput: number = parseInt(this.hazszamInputStr, 10);
    const keresettTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput);
    const balSzomszedTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput + 2);
    const jobbSzomszedTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput - 2);
    let lehetsegesSzinek: string[] = ["A", "B", "C", "D"];
    if (keresettTelek.length !== 0) {
      // ha van telek, aminek a kerítését festeni kell
      lehetsegesSzinek = lehetsegesSzinek.filter(x => x !== keresettTelek[0].keritesSzine);
    } else return "Nincs ilyen házszám!";
    // ha van bal szomszéd:
    if (balSzomszedTelek.length !== 0) {
      lehetsegesSzinek = lehetsegesSzinek.filter(x => x !== balSzomszedTelek[0].keritesSzine);
    }
    // ha van jobb szomszéd:
    if (jobbSzomszedTelek.length !== 0) {
      lehetsegesSzinek = lehetsegesSzinek.filter(x => x !== jobbSzomszedTelek[0].keritesSzine);
    }
    return lehetsegesSzinek[0];
  }
}
</script>

<style>
body{
  overflow: hidden;
}
#app {
  font-family: Courier;
}

#megoldas {
  background-color: lightgrey;
  padding: 0px 10px;
  border-radius: 10px;
  max-width: 600px;
}

a {
  text-decoration: none;
  padding-left: 10px;
}

input[type="number"] {
  background-color: lightgray;
}

pre {
  font-size: 1.1em;
  margin: 0;
}
</style>
