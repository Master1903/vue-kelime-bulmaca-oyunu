<template>
  <div id="app" class="container mt-4">
    <div class="card mb-4" v-if="tamamlandi">
      <div class="card-body">Tebrikler yarışmayı {{this.puan}} ile tamamladınız</div>
    </div>

    <div class="card" v-if="!mevcutSoru">
      <div class="card-header">
        <h5 class="mb-0">Kelime Oyunu Yarışmasına</h5>
      </div>
      <div class="card-body">Yarışmaya Başlamak için Butona basın</div>
      <div class="card-footer">
        <button class="btn btn-primary" @click="basla">Başla</button>
      </div>
    </div>
    <div class="card" v-else>
      <div class="card-header">
        <h3 class="mb-0">{{mevcutSoru.soru}}</h3>
      </div>
      <div class="card-body">
        <div class="d-flex">
          <Harf :deger="harf.harf" :acik="harf.acildi" v-for="(harf,i) in harfler" :key="'harf'+i" />
        </div>
      </div>
      <div class="card-footer">
        <button class="btn btn-secondary" @click="harfVer">Harf Ver</button>
      </div>
      <div class="card-footer">
        <div class="d-flex">
          <div class="mr-4">Toplam Puan : {{puan}}</div>
          <div class="mr-4">
            <kbd>{{this.kalanSure}} saniye</kbd>
          </div>
          <div>Harf Puanı : {{harfPuan}}</div>
        </div>
      </div>
      <div class="card-footer">
        <div class="input-group mb-3">
          <input
            type="text"
            class="form-control"
            v-model="yarismaciCevap"
            placeholder="Cevabınız"
            aria-describedby="basic-addon2"
            @keyup="yarismaciCevap = yarismaciCevap.toLocaleUpperCase('tr')"
          />
          <div class="input-group-append">
            <button class="btn btn-success" type="button" @click="cevapVer">Cevap Ver</button>
          </div>
        </div>
      </div>
      <div class="card-footer" :class="mesaj.class" v-if="mesaj.metin">{{mesaj.metin}}</div>
    </div>
  </div>
</template>

<script>
import Harf from "@/components/Harf.vue";

export default {
  name: "App",
  components: { Harf },
  data() {
    return {
      sorular: [
        {
          soru: "Siyah ile aynı anlama gelen bir kelime",
          cevap: "KARA",
          soruldu: false
        },
        {
          soru: "Sık kullanılan bir isim",
          cevap: "AHMET",
          soruldu: false
        },
        {
          soru: "Türkiyenin başkenti",
          cevap: "ANKARA",
          soruldu: false
        },
        {
          soru: "Karadenizde bir ilimiz",
          cevap: "TRABZON",
          soruldu: false
        }
      ],
      mevcutSoru: null,
      puan: 0,
      harfPuan: 0,
      harfler: [],
      yarismaciCevap: "",
      tamamlandi: false,
      sure: null,
      kalanSure: 0,
      mesaj: {
        metin: "",
        class: null,
        sure: null
      }
    };
  },
  methods: {
    basla() {
      this.tamamlandi = false;
      this.mevcutSoru = null;
      this.puan = 0;
      this.kalanSure = 240; // 240 saniye
      this.sure = setInterval(() => {
        this.kalanSure--;
        if (this.kalanSure === 0) this.bitir();
      }, 1000);
      this.sorular.map(x => {
        x.soruldu = false;
      });
      this.soruVer();
      this.mesajGoster("İyi Yarışmalar");
    },
    bitir() {
      clearInterval(this.sure);
      this.tamamlandi = true;
      this.mevcutSoru = null;
    },
    soruVer() {
      this.yarismaciCevap = "";
      this.mevcutSoru = this.sorular.find(x => !x.soruldu); // sorulmayan sorular
      if (!this.mevcutSoru) {
        this.bitir();
        return;
      }
      this.harfler = [];
      this.mevcutSoru.cevap.split("").map(x => {
        this.harfler.push({
          harf: x,
          acildi: false
        });
      });
      this.harfPuan = this.mevcutSoru.cevap.length * 100;
      this.mevcutSoru.soruldu = true;
    },
    harfVer() {
      if (this.harfPuan <= 100) return;
      let rastgeleHarfIndex = Math.floor(Math.random() * this.harfler.length);
      let bulunanHarf = this.harfler[rastgeleHarfIndex];
      while (bulunanHarf.acildi) {
        rastgeleHarfIndex = Math.floor(Math.random() * this.harfler.length);
        bulunanHarf = this.harfler[rastgeleHarfIndex];
      }
      bulunanHarf.acildi = true;
      this.harfPuan -= 100;
    },
    cevapVer() {
      if (!this.yarismaciCevap.length) {
        return;
      }
      if (this.yarismaciCevap.length !== this.harfler.length) {
        this.mesajGoster(
          "Harf sayınız ile cevabınızdaki harf sayıları tutmuyor. Cevabınızı tekrar gözden geçirin",
          "uyari"
        );
        return;
      }
      if (this.yarismaciCevap === this.mevcutSoru.cevap) {
        this.puan += this.harfPuan;
        this.mesajGoster("Tebrikler Doğru Bildiniz", "basari");
      } else {
        this.puan -= this.harfPuan;
        this.mesaj.class = "";
        this.mesajGoster(
          `Yanlış Cevap, doğrusu ${this.mevcutSoru.cevap} olmalıydı!`,
          "hata"
        );
      }
      this.soruVer();
    },
    mesajGoster(mes, tip) {
      this.mesaj.metin = mes;
      switch (tip) {
        case "hata":
          this.mesaj.class = "bg-danger text-white";
          break;
        case "basari":
          this.mesaj.class = "bg-success text-white";
          break;
        case "uyari":
          this.mesaj.class = "bg-warning text-white";
          break;

        default:
          this.mesaj.class = "bg-info text-white";
      }

      if (this.mesaj.sure) {
        clearTimeout(this.mesaj.sure);
        this.mesaj.sure = null;
      }

      this.mesaj.sure = setTimeout(() => {
        this.mesaj.metin = null;
      }, 3000);
    }
  }
};
</script>


