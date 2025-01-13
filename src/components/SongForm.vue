<script setup>
import { ref } from 'vue';
import { languages } from '@/const/Languages';
const props = defineProps(['song'])
let title = ref(props.song.title);
let titleYomi = ref('');
let lyrics = ref('');
let audioFileLink = ref('');
let selectedLanguages = ref([]);

</script>

<template>
    <div>
        <form class="w3-container">
            <label><b>楽曲名</b></label>
            <input v-model="title" type="text" placeholder="楽曲名" class="w3-input" />
            <label><b>楽曲名ヨミ</b></label>
            <input v-model="titleYomi" type="text" placeholder="ガッキョクメイ" class="w3-input" />
            <label><b>歌詞</b></label>
            <textarea v-model="lyrics" type="text" maxlength="1000" placeholder="歌詞を入力"
                class="w3-input">{{ lyrics }}</textarea>
            
                <p :class='lyrics.length==1000?"w3-pale-red":""'><span>{{ lyrics.length }}/1000文字</span></p>
                
            
            <label><b>音源リンク</b></label>
            <input v-model="audioFileLink" type="url" class="w3-input" />
            <label><b>翻訳先言語</b></label>
            
            <div>
                <div v-for="language in languages">
                    <input type="checkbox" :id="language.id" :value="language.id" v-model="selectedLanguages" />
                    <label :for="language.id">{{ language.value }}</label><br>
                </div>
            </div>
            <label><b>金額</b></label>
            <p>¥{{ selectedLanguages.length * 3500 }}</p>
        </form>
    </div>

</template>
