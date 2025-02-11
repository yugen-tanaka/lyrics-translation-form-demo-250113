<script setup>
import { ref } from 'vue';
import SongListView from './SongListView.vue';
let name = ref("");
let email = ref("");
let emailcheck = ref("");
let songs = ref(JSON.parse(localStorage.getItem("songs")) || []);
let confirmButtonDisable = ref(true);

</script>

<template>
    <header class="w3-container w3-black">
        <div>

            <h1 class="w3-text-lime"> <span><img class="w3-round w3-image" src="../assets/musicMarketingLab.webp"
                        alt="Music Marketing Labのロゴ" style="width:100%;max-width:150px"></span>
                <b>歌詞多言語翻訳サービス</b>
            
            </h1>
           
        </div>
        <span class="w3-text-red">※このフォームはデモ版です。</span>
    </header>
    <div>
        <h3>ご連絡先</h3>
        <form id="contact" class="w3-container">

            <label>お名前</label>
            <input v-model="name" type="text"  class="w3-input w3-border w3-round" placeholder="楽簿太郎" required />

            <label>メールアドレス</label>
            <input v-model="email" type="email"  class="w3-input w3-border w3-round" placeholder="taro.lab@example.com" required />
            <label>メールアドレス(確認用)</label>
            <input v-model="emailcheck" type="email"  class="w3-input w3-border w3-round" required />
        </form>
        <SongListView></SongListView>
        <button type="submit" form="contact" class="w3-btn w3-round-large w3-black w3-text-lime"><b>確認画面へ進む</b></button>
    </div>
</template>