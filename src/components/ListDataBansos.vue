<template>
    <div class="list-container">
      <h2>Daftar Data Bantuan Sosial</h2>
      <table>
        <thead>
          <tr>
            <th>Nama</th>
            <th>NIK</th>
            <th>Details</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(data, index) in dataList" :key="index">
            <td>{{ data.nama }}</td>
            <td>{{ data.nik }}</td>
            <td><button @click="viewDetail(index)">Lihat Detail</button></td>
          </tr>
        </tbody>
      </table>
      <button @click="goBack">Back to Home</button>
    </div>
  </template>
  
  <script>
  export default {

    data() {
      return {
        dataList: JSON.parse(localStorage.getItem('dataList')) || []
      };
    },
    methods: {
      viewDetail(index) {
        this.$router.push({ name: 'DetailDataBansos', params: { index } });
      },
      goBack() {
        this.$router.push('/');
      }
    }
  };
  </script>

<style scoped>
@import '../assets/styles.css';
</style>
