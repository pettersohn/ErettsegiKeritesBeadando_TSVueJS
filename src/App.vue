<template>
  <div id="app">
    <b-navbar id="navbar" toggleable="lg" type="dark" variant="primary">
      <b-navbar-brand href="#">Kerítés feladat</b-navbar-brand>
      <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>
      <b-collapse id="nav-collapse" is-nav>
        <b-navbar-nav>
          <b-nav-item href="kerites.txt">kerites.txt</b-nav-item>
          <b-nav-item href="Kerítés_fel.pdf">Feladat.pdf</b-nav-item>
          <b-nav-item href="Kerítés_jav.pdf">Javítási.pdf</b-nav-item>
          <b-nav-item href="https://github.com/pettersohn/ErettsegiKeritesBeadando_TSVueJS">GitHub</b-nav-item>
          <b-nav-item href="https://kerites-pettersohn.netlify.com/">Netlify</b-nav-item>
        </b-navbar-nav>
      </b-collapse>
    </b-navbar>
    <TxtReader
      label="Forrásfájl (kerites.txt):"
      placeholder="Kérem töltse be a forrásfájlt!"
      @load="txtSorai = $event"
    />
    <div v-if="mutat" id="megoldas" style="float:left">
      <b-tabs id="tabok" content-class="mt-4" align="center">
        <b-tab title="2.feladat">
          <p>Az eladott telkek száma: {{ telkek.length }}</p>
        </b-tab>
        <b-tab title="3.feladat">
          <p>
            A {{ utolsoTelek.oldal }} oldalon adták el az utolsó telket<br />
            Az utolsó telek házszáma: {{ utolsoTelek.hazSzama }}
          </p>
        </b-tab>
        <b-tab title="4.feladat">
          <p>A szomszédossal egyezik a kerítés színe: {{ ugyanOlyanSzinuKerites }}</p>
        </b-tab>
        <b-tab title="5.feladat">
          <p>
            Adjon meg egy házszámot!
            <input v-model="hazszamInputStr" type="number" min="1" max="117" /><br />
            A kerírés színe / állapota: {{ keritesSzineAllapota }}<br />
            Egy lehetséges festési szín: {{ lehetsegesFestesiSzin }}
          </p>
        </b-tab>
      </b-tabs>
    </div>
    <div v-if="mutat" id="egyebek" style="float:right">
      <div class="overflow-auto">
        <pre>
utcakep.txt fájl:
{{ utcakep.trim() }}
      </pre
        >
      </div>
      <div class="overflow-auto">
        <pre>
kerites.txt fájl:
{{ txtSorai.trim() }}
      </pre
        >
      </div>
    </div>
    <b-button v-if="mutat" pill variant="success" class="savebutton mx-auto">
      <TxtWriter title="utcakep.txt állomány mentése" :content="utcakep" filename="utcakep.txt" />
    </b-button>
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
pre {
  color: white;
}

#megoldas {
  background-color: #c84d5f;
  color: #ddc23f;
  padding: 0px 10px;
  border-radius: 10px;
  max-width: 550px;
  height: 200px;
  text-align: center;
  margin: 20px;
}

#egyebek {
  background-color: #c84d5f;
  color: #ddc23f;
  padding: 0px 10px;
  border-radius: 10px;
  max-width: 550px;
  height: 200px;
  text-align: center;
  margin: 20px;
}
b-tab {
  margin-top: 155px;
}

b-tab title {
  color: #ddc23f;
}

a {
  text-decoration: none;
  padding-left: 10px;
}

input[type="number"] {
  background-color: white;
  border: 2px gold solid;
  border-radius: 10px;
  text-align: center;
}

.savebutton {
  background-color: #c84d5f;
  border: 2px solid red;
  color: gold;
  position: center;
  display: flex;
}
</style>
