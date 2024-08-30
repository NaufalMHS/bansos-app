<template>
  <div class="form-container">
    <h1>Formulir Penerima Bantuan Sosial</h1>
    <form @submit.prevent="submitForm">
      <div>
        <label for="nama">Nama</label>
        <input type="text" v-model="formData.nama" required />
      </div>
      <div>
        <label for="nik">NIK</label>
        <input type="number" v-model="formData.nik" required />
      </div>
      <div>
        <label for="noKartuKeluarga">Nomor Kartu Keluarga</label>
        <input type="number" v-model="formData.noKartuKeluarga" required />
      </div>
      <div>
        <label for="fotoKTP">Foto KTP</label>
        <input type="file" @change="handleFileChange('fotoKTP', $event)" accept="image/*" />
      </div>
      <div>
        <label for="fotoKK">Foto Kartu Keluarga</label>
        <input type="file" @change="handleFileChange('fotoKK', $event)" accept="image/*" />
      </div>
      <div>
        <label for="umur">Umur</label>
        <input type="number" v-model="formData.umur" min="25" required />
      </div>
      <div>
        <label for="jenisKelamin">Jenis Kelamin</label>
        <select v-model="formData.jenisKelamin" required>
          <option value="Laki-laki">Laki-laki</option>
          <option value="Perempuan">Perempuan</option>
        </select>
      </div>
      <div>
        <label for="provinsi">Provinsi</label>
        <select v-model="formData.provinsi" @change="loadKabupaten" required>
          <option v-for="prov in provinsiList" :key="prov.id" :value="prov.id">{{ prov.name }}</option>
        </select>
      </div>
      <div>
        <label for="kabupaten">Kabupaten/Kota</label>
        <select v-model="formData.kabupaten" @change="loadKecamatan" required>
          <option v-for="kab in kabupatenList" :key="kab.id" :value="kab.id">{{ kab.name }}</option>
        </select>
      </div>
      <div>
        <label for="kecamatan">Kecamatan</label>
        <select v-model="formData.kecamatan" @change="loadKelurahan" required>
          <option v-for="kec in kecamatanList" :key="kec.id" :value="kec.id">{{ kec.name }}</option>
        </select>
      </div>
      <div>
        <label for="kelurahan">Kelurahan/Desa</label>
        <select v-model="formData.kelurahan" required>
          <option v-for="kel in kelurahanList" :key="kel.id" :value="kel.id">{{ kel.name }}</option>
        </select>
      </div>
      <div>
        <label for="alamat">Alamat</label>
        <input type="text" v-model="formData.alamat" maxlength="255" required />
      </div>
      <div>
        <label for="rt">RT</label>
        <input type="number" v-model="formData.rt" required />
      </div>
      <div>
        <label for="rw">RW</label>
        <input type="number" v-model="formData.rw" required />
      </div>
      <div>
        <label for="penghasilanSebelumPandemi">Penghasilan Sebelum Pandemi</label>
        <input type="number" v-model="formData.penghasilanSebelumPandemi" required />
      </div>
      <div>
        <label for="penghasilanSetelahPandemi">Penghasilan Setelah Pandemi</label>
        <input type="number" v-model="formData.penghasilanSetelahPandemi" required />
      </div>
      <div>
        <label for="alasanMembutuhkan">Alasan Membutuhkan Bantuan</label>
        <select v-model="formData.alasanMembutuhkan" @change="checkAlasanLainnya" required>
          <option value="Kehilangan pekerjaan">Kehilangan pekerjaan</option>
          <option value="Kepala keluarga">Kepala keluarga</option>
          <option value="Tergolong fakir/miskin">Tergolong fakir/miskin</option>
          <option value="Lainnya">Lainnya: … (bisa diisi sendiri)</option>
        </select>
      </div>
      <div v-if="showAlasanLainnya">
        <label for="alasanLainnya">Tulis Alasan Lainnya</label>
        <input type="text" v-model="formData.alasanLainnya" placeholder="Tulis alasan di sini" />
      </div>
      <div>
        <label>
          <input type="checkbox" v-model="confirm" required />
          Saya menyatakan bahwa data yang diisikan adalah benar dan siap mempertanggungjawabkan apabila ditemukan ketidaksesuaian dalam data tersebut.
        </label>
      </div>
      <button type="submit">Submit</button>
      <button type="button" @click="goToPreview" v-if="formData">Lihat List Data</button>
    </form>
  </div>
</template>


<script>
import { api } from '../services/api';

export default {
  data() {
    return {
      formData: {
        nama: '',
        nik: '',
        noKartuKeluarga: '',
        fotoKTP: null,
        fotoKK: null,
        umur: '',
        jenisKelamin: '',
        provinsi: '',
        kabupaten: '',
        kecamatan: '',
        kelurahan: '',
        alamat: '',
        rt: '',
        rw: '',
        penghasilanSebelumPandemi: '',
        penghasilanSetelahPandemi: '',
        alasanMembutuhkan: '',
        alasanLainnya: ''
      },
      provinsiList: [],
      kabupatenList: [],
      kecamatanList: [],
      kelurahanList: [],
      confirm: false,
      showAlasanLainnya: false
    };
  },
  created() {
    this.loadProvinsi();
    this.loadFromLocalStorage();
  },
  watch: {
    formData: {
      handler() {
        this.saveToLocalStorage();
      },
      deep: true
    }
  },
  methods: {
    async submitForm() {
    if (this.confirm) {
      try {
        const { isSuccess, responseTime } = await this.simulateBackendService();
        if (isSuccess) {
          const existingData = JSON.parse(localStorage.getItem('dataList')) || [];
          existingData.push(this.formData);
          localStorage.setItem('dataList', JSON.stringify(existingData));
          alert(`Data berhasil disimpan dalam waktu ${responseTime / 1000} detik!`);
          this.$router.push('/list'); 
        } else {
          alert(`Gagal menyimpan data setelah ${responseTime / 1000} detik, silakan coba lagi.`);
        }
      } catch (error) {
        alert('Terjadi kesalahan saat menyimpan data.');
      }
    } else {
      alert('Harap centang pernyataan kebenaran data.');
    }
  },
  simulateBackendService() {
    return new Promise((resolve) => {
      const responseTime = Math.floor(Math.random() * 1000) + 1000;

      setTimeout(() => {        if (responseTime < 1500) {
          resolve({ isSuccess: true, responseTime }); 
        } else {
          resolve({ isSuccess: false, responseTime });
        }
      }, responseTime);
    });
  },
  goToPreview() {
    localStorage.setItem('formData', JSON.stringify(this.formData));
    this.$router.push('/list');
  },
    async loadProvinsi() {
      try {
        const response = await api.loadProvinsi();
        this.provinsiList = response.data;
      } catch (error) {
        console.error('Gagal memuat data provinsi:', error);
      }
    },
    async loadKabupaten() {
      try {
        const response = await api.loadKabupaten(this.formData.provinsi);
        this.kabupatenList = response.data;
      } catch (error) {
        console.error('Gagal memuat data kabupaten:', error);
      }
    },
    async loadKecamatan() {
      try {
        const response = await api.loadKecamatan(this.formData.kabupaten);
        this.kecamatanList = response.data;
      } catch (error) {
        console.error('Gagal memuat data kecamatan:', error);
      }
    },
    async loadKelurahan() {
      try {
        const response = await api.loadKelurahan(this.formData.kecamatan);
        this.kelurahanList = response.data;
      } catch (error) {
        console.error('Gagal memuat data kelurahan:', error);
      }
    },
    handleFileChange(field, event) {
    const file = event.target.files[0];
    if (file && file.size <= 2 * 1024 * 1024) {
      const reader = new FileReader();
      reader.onloadend = () => {
        this.formData[field] = reader.result; 
        this.$emit('form-updated', this.formData);
      };
      reader.readAsDataURL(file); 
    } else {
      alert('File terlalu besar atau tidak valid!');
    }
    },
    checkAlasanLainnya() {
      this.showAlasanLainnya = this.formData.alasanMembutuhkan === 'Lainnya';
      if (!this.showAlasanLainnya) {
        this.formData.alasanLainnya = '';
      }
    },
    saveToLocalStorage() {
      localStorage.setItem('formData', JSON.stringify(this.formData));
    },
    loadFromLocalStorage() {
      const savedData = localStorage.getItem('formData');
      if (savedData) {
        this.formData = JSON.parse(savedData);
      }
    }
  }
};
</script>


<style scoped>
@import '../assets/styles.css';
</style>
