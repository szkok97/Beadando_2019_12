<template>
  <div id="app">
    <b-navbar toggleable="lg" type="light" variant="background" class="menuk">
      <b-navbar-brand href="#">Kerítés feladat</b-navbar-brand>
      <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>
      <b-collapse id="nav-collapse" is-nav>
        <b-navbar-nav>
          <b-nav-item class="menupontok" href="kerites.txt">kerites.txt</b-nav-item>
          <b-nav-item class="menupontok" href="Kerítés_fel.pdf">Feladat.pdf</b-nav-item>
          <b-nav-item class="menupontok" href="Kerítés_jav.pdf">Javítási.pdf</b-nav-item>
          <b-nav-item class="menupontok" href="https://github.com/">GitHub</b-nav-item>
          <b-nav-item class="menupontok" href="https://github.com/szkok97/Beadando_2019_12">SDK</b-nav-item>
        </b-navbar-nav>
      </b-collapse>
    </b-navbar>
    <div class="container-fluid">
      <div class="sor">
        <TxtReader label="Forrás (kerites.txt):" placeholder="Bedobó mező" @load="txtSorai = $event" />
        <div v-if="mutat" id="megoldas" class="oszlopegy">
          <div class="belsomenu">
            <p>2. feladat<br />Az eladott telkek száma: {{ telkek.length }}</p>
          </div>
          <div class="belsomenu">
            <p>
              3. feladat<br />A {{ utolsoTelek.oldal }} oldalon adták el az utolsó telket<br />
              Az utolsó telek házszáma: {{ utolsoTelek.hazSzama }}
            </p>
          </div>
          <div class="belsomenu">
            <p>4. feladat<br />A szomszédossal egyezik a kerítés színe: {{ ugyanOlyanSzinuKerites }}</p>
          </div>
          <div class="belsomenu">
            <p>
              5. feladat<br />Adjon meg egy házszámot!
              <input v-model="hazszamInputStr" type="number" min="1" max="117" /><br />
              A kerírés színe / állapota: {{ keritesSzineAllapota }}<br />
              Egy lehetséges festési szín: {{ lehetsegesFestesiSzin }}
            </p>
          </div>
        </div>
        <div>
          <p v-if="mutat">
            <TxtWriter title="utcakep.txt állomány mentése" :content="utcakep" filename="utcakep.txt" class="gomb" />
          </p>
        </div>
      </div>
      <!-- Megoldás DIV -->

      <!-- Nem a feladat része : -->
      <div v-if="mutat" id="egyebek">
        <p>utcakep.txt fájl:</p>
        <pre
          >{{ utcakep }}  
         </pre
        >
      </div>
      <div v-if="mutat" id="egyebek">
        <p>kerites.txt fájl:</p>
        <pre
          >{{ txtSorai }}
        </pre>
      </div>
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
.gomb {
  background-color: cornflowerblue;
  margin: 10px;
}
#app {
  font-family: Courier;
}

#megoldas {
  background-color: darkblue;
  padding: 15px 10px;
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
  background-color: cornflowerblue;
  border-radius: 10px;
}
.menupontok:hover {
  background-color: yellow;
}
.menuk {
  background-color: cornflowerblue;
}
.belsomenu {
  background-color: lightblue;
  padding: 0px 10px;
  border-radius: 10px;
  max-width: 600px;
  margin-bottom: 5px;
}
</style>
