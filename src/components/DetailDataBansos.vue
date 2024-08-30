<!-- DetailDataBansos.vue -->
<template>
    <div class="detail-container">
      <h2>Detail Data Bantuan Sosial</h2>
      <table>
        <thead>
          <tr>
            <th>Field</th>
            <th>Value</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>Nama</td>
            <td>{{ detailData.nama }}</td>
          </tr>
          <tr>
            <td>NIK</td>
            <td>{{ detailData.nik }}</td>
          </tr>
          <tr>
            <td>Nomor Kartu Keluarga</td>
            <td>{{ detailData.noKartuKeluarga }}</td>
          </tr>
          <tr>
            <td>Foto KTP</td>
            <td>
              <div class="image-container" @click="openModal('fotoKTP')">
                <img v-if="detailData.fotoKTP" :src="detailData.fotoKTP" alt="Foto KTP" class="preview-image" />
              </div>
            </td>
          </tr>
          <tr>
            <td>Foto Kartu Keluarga</td>
            <td>
              <div class="image-container" @click="openModal('fotoKK')">
                <img v-if="detailData.fotoKK" :src="detailData.fotoKK" alt="Foto Kartu Keluarga" class="preview-image" />
              </div>
            </td>
          </tr>
          <tr>
            <td>Umur</td>
            <td>{{ detailData.umur }}</td>
          </tr>
          <tr>
            <td>Jenis Kelamin</td>
            <td>{{ detailData.jenisKelamin }}</td>
          </tr>
          <tr>
            <td>Provinsi</td>
            <td>{{ getWilayahName('provinsi', detailData.provinsi) }}</td>
          </tr>
          <tr>
            <td>Kabupaten/Kota</td>
            <td>{{ getWilayahName('kabupaten', detailData.kabupaten) }}</td>
          </tr>
          <tr>
            <td>Kecamatan</td>
            <td>{{ getWilayahName('kecamatan', detailData.kecamatan) }}</td>
          </tr>
          <tr>
            <td>Kelurahan/Desa</td>
            <td>{{ getWilayahName('kelurahan', detailData.kelurahan) }}</td>
          </tr>
          <tr>
            <td>Alamat</td>
            <td>{{ detailData.alamat }}</td>
          </tr>
          <tr>
            <td>RT</td>
            <td>{{ detailData.rt }}</td>
          </tr>
          <tr>
            <td>RW</td>
            <td>{{ detailData.rw }}</td>
          </tr>
          <tr>
            <td>Penghasilan Sebelum Pandemi</td>
            <td>{{ detailData.penghasilanSebelumPandemi }}</td>
          </tr>
          <tr>
            <td>Penghasilan Setelah Pandemi</td>
            <td>{{ detailData.penghasilanSetelahPandemi }}</td>
          </tr>
          <tr>
            <td>Alasan Membutuhkan Bantuan</td>
            <td>{{ detailData.alasanMembutuhkan }}<span v-if="detailData.alasanMembutuhkan === 'Lainnya'">: {{ detailData.alasanLainnya }}</span></td>
          </tr>
        </tbody>
      </table>
      <button @click="goBack">Kembali ke List</button>
      
      <!-- Modal -->
      <div v-if="isModalOpen" class="modal" @click.self="closeModal">
        <div class="modal-content">
          <span class="close" @click="closeModal">&times;</span>
          <img :src="selectedImage" class="modal-image" />
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import { api } from '../services/api';
  
  export default {
    data() {
      return {
        detailData: {},
        provinsiList: [],
        kabupatenList: [],
        kecamatanList: [],
        kelurahanList: [],
        isModalOpen: false,
        selectedImage: ''
      };
    },
    async created() {
      this.loadDetailData();
      await this.loadAllWilayah();
    },
    methods: {
      async loadDetailData() {
        const index = parseInt(this.$route.params.index, 10);
        const dataList = JSON.parse(localStorage.getItem('dataList')) || [];
        this.detailData = dataList[index] || {};
      },
      async loadAllWilayah() {
        try {
          const [provinsiRes, kabupatenRes, kecamatanRes, kelurahanRes] = await Promise.all([
            api.loadProvinsi(),
            api.loadKabupaten(this.detailData.provinsi),
            api.loadKecamatan(this.detailData.kabupaten),
            api.loadKelurahan(this.detailData.kecamatan)
          ]);
  
          this.provinsiList = provinsiRes.data;
          this.kabupatenList = kabupatenRes.data;
          this.kecamatanList = kecamatanRes.data;
          this.kelurahanList = kelurahanRes.data;
        } catch (error) {
          console.error('Gagal memuat data wilayah:', error);
        }
      },
      getWilayahName(type, id) {
        let list = [];
        switch (type) {
          case 'provinsi':
            list = this.provinsiList;
            break;
          case 'kabupaten':
            list = this.kabupatenList;
            break;
          case 'kecamatan':
            list = this.kecamatanList;
            break;
          case 'kelurahan':
            list = this.kelurahanList;
            break;
        }
        const item = list.find(item => item.id === id);
        return item ? item.name : 'Tidak diketahui';
      },
      openModal(imageType) {
        this.selectedImage = this.detailData[imageType];
        this.isModalOpen = true;
      },
      closeModal() {
        this.isModalOpen = false;
        this.selectedImage = '';
      },
      goBack() {
        this.$router.push('/list');
      }
    }
  };
  </script>
  

  

  <style scoped>
@import '../assets/styles.css';
</style>
