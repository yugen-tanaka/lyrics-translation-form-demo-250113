<script setup>
import { ref, computed, onMounted } from 'vue';
import { languages } from '@/const/Languages';
import SongForm from './SongForm.vue';
import { errorMessages } from 'vue/compiler-sfc';

let songs = ref(JSON.parse(localStorage.getItem("songs")) || []);
const songsLimit = 20;
const srtFee = 1000;
const languageFee = 3500;

const isErrMsg = ref(false);
const errMsg = ref('');

let isEditing = ref(false);
let artistName = ref('');
let title = ref('');
let titleYomi = ref('');
let lyrics = ref('');
let audioFileLink = ref('');
let selectedLanguages = ref([]);
let needSRT = ref(false);
const audioFile = ref(null);
let audioSrc = ref("");


let deadline = ref(new Date());
let isLimitModal = ref(false);
let isDeleteModal = ref(false);
let selectedSong = ref();
const lyricsFileContent = ref('');
let audioKey = ref(0);


onMounted(() => {
    const today = new Date();
    const sevenDaysAfter = new Date(today);
    sevenDaysAfter.setDate(today.getDate() + 7);
    deadline.value = sevenDaysAfter;
});
const formattedDate = computed(() => {

    return deadline.value.toLocaleDateString('ja-JP', { timeZone: 'JST' });
});

function addSong() {
    if (songs.value.length >= songsLimit) {
        isLimitModal.value = true;
        return;
    };
    const today = new Date();
    let sevenDaysAfter = new Date();
    sevenDaysAfter.setDate(today.getDate() + 7);
    deadline.value = sevenDaysAfter;
    console.log(sevenDaysAfter.toLocaleDateString('ja-JP', { timeZone: 'JST' }));
    console.log("addSong()");
    const newSong = {
        id: crypto.randomUUID(),
        title: "",
        titleYomi: "",
        lyrics: "",
        audioFileLink: "",
        languages: [],
        price: 0,
        deadline: deadline.value,
        needSRT: false,
        isEditing: true
    };
    title.value = newSong.title;
    titleYomi.value = newSong.titleYomi;
    lyrics.value = newSong.lyrics;
    selectedLanguages.value = newSong.languages;
    needSRT.value = newSong.needSRT;
    songs.value.push(newSong);
    isEditing.value = true;
}

function editSong(id) {
    for (const song of songs.value) {
        if (song.isEditing) {
            if (song.title == "" || song.titleYomi == "" || song.lyrics == "" || song.languages.length > 0) {
                isErrMsg.vaule = true;
                errMsg.value = '必須項目が未入力です';
                return;
            }


        }
        if (song.id == id) {
            isEditing.value = true;
            song.isEditing = true;
            title.value = song.title;
            titleYomi.value = song.titleYomi;
            lyrics.value = song.lyrics;
            selectedLanguages.value = song.languages;
            needSRT.value = song.needSRT;
        } else {
            song.isEditing = false;
        }
    };
}
function cancelEditing() {


    songs.value.forEach(song => {
        song.isEditing = false;
        //songs.value = localStorage.getItem("songs");
    });
    isEditing.value = false;

    isErrMsg.value = false;
    songs.value = JSON.parse(localStorage.getItem("songs") || []);
}
function finishEditingSong(id) {

    for (const song of songs.value) {
        if (song.id == id) {
            if (title.value == "" || titleYomi.value == "" || lyrics.value == "" || selectedLanguages.value.length == 0) {
                isErrMsg.value = true;
                errMsg.value = '必須項目が未入力です';
                return;
            }
            isErrMsg.value = false;
            const updatedSong = {
                id: id,
                title: title.value,
                titleYomi: titleYomi.value,
                lyrics: lyrics.value,
                audioFileLink: audioFileLink.value,
                languages: selectedLanguages.value,
                price: (languageFee * selectedLanguages.value.length + (needSRT.value ? srtFee : 0)),
                deadline: deadline.value,
                needSRT: needSRT.value,
                isEditing: false
            }
            const index = songs.value.findIndex((song) => song.id === id);
            if (index !== -1) {
                songs.value[index] = updatedSong;
            } else {
                console.log("指定されたインデックスのSongが見つかりませんでした。", id)
            }
            isEditing.value = false;
            localStorage.setItem("songs", JSON.stringify(songs.value));

        };
    };
}
function closeLimitModal() {
    isLimitModal.value = false;
}

function showDeleteModal(song) {
    selectedSong.value = song;
    isDeleteModal.value = true;
}
function closeDeleteModal() {
    selectedSong.value = undefined;
    isDeleteModal.value = false;
}
function deleteSong(id) {
    const index = songs.value.findIndex(song => song.id == id);
    songs.value.splice(index, 1);
    localStorage.setItem("songs", JSON.stringify(songs.value));
    isDeleteModal.value = false;
}
function handleLyricsFile(event) {
    const file = event.target.files[0]; // 選択されたファイルを取得
if (!file) {
    return;
};
if (file && file.type === "text/plain") {
  const reader = new FileReader(); // FileReader API を使用
  reader.onload = (e) => {
    const result = e.target.result;
    if (result.length>1000) {
        alert("文字数が1000文字を超えています。");
        return;
    }
    lyrics.value = e.target.result; // ファイル内容を格納
    console.log(lyrics.value);
  };
  reader.readAsText(file); // テキストとしてファイルを読み取る
} else {
  alert("有効なテキストファイルを選択してください (.txt)");
}
}

function handleAudioUpload(event) {
  const selectedFile = event.target.files[0];
  if (!selectedFile) {
    return;
};
  // ファイルが選択されていない場合
  if (!selectedFile) {
    errMsg.value = "ファイルを選択してください。";
    alert(errMsg.value);
    return;
  }

  // ファイル形式のチェック
  if (selectedFile.type !== "audio/mpeg") {
    errMsg.value = "MP3形式の音源ファイルのみアップロード可能です。";
    alert(errMsg.value);
    return;
  }

  // ファイルサイズのチェック（例: 10MB制限）
  const maxSizeMB = 10;
  if (selectedFile.size > maxSizeMB * 1024 * 1024) {
    errMsg.value = `ファイルサイズは最大${maxSizeMB}MBまでです。`;
    alert(errMsg.value);
    return;
  }

  // 問題がなければデータをセット
  audioFile.value = selectedFile;
  errMsg.value = "";

  // FileReader を使って音声ファイルのURLを生成
  const reader = new FileReader();
  reader.onload = (e) => {
    audioSrc.value = e.target.result.replace("data:audio/mpeg", "data:audio/mp3"); 
    console.log("File: loaded (audioSrc):)",audioSrc.value);
    audioKey.value += 1;
  };
  reader.readAsDataURL(selectedFile);
  
}
</script>

<template>
    <h3>楽曲リスト</h3>
    <button @click="addSong" class="w3-btn w3-round-large w3-lime " :disabled="isEditing">+ 曲を追加</button>

    <div>
        <table class="w3-table w3-bordered">
            <thead class="w3-light-gray ">
                <th>曲名</th>
                <th>言語</th>
                <th>金額</th>
                <th>編集</th>
                <th>削除</th>
            </thead>
            <tbody>
            <tr v-for="song in songs" :key="song.id">
                <template v-if="!song.isEditing">
                    <td>{{ song.title }}</td>
                    <td>{{ song.languages.map((a) => a.value).join(", ") }}</td>
                    <td>¥{{ song.price }}</td>
                    <td><button @click="editSong(song.id)" :disabled="isEditing"
                            class="w3-button w3-round-large w3-lime">編集</button>
                    </td>
                    <td><button @click="showDeleteModal(song)" :disabled="isEditing"
                            class="w3-button w3-round-large w3-light-gray w3-text-red">削除</button>
                    </td>
                </template>
                <template v-else>
                    <td colspan="4">

                        <div class="w3-display-container" style="width: 100%;">

                            <form class="w3-container">
                                <section>
                                    <label for="artistName"><b>アーティスト名</b><span class="w3-text-red">*</span></label>
                                    <input class="w3-input w3-border w3-round" id="artistName" v-model="artistName" type="text" placeholder="アーティスト名" />
                                    <label for="title"><b>楽曲名</b><span class="w3-text-red">*</span></label>
                                    <input id="title" v-model="title" type="text" placeholder="楽曲名"
                                        class="w3-input w3-border w3-round" />
                                    <label for="titleYomi"><b>楽曲名ヨミ</b><span class="w3-text-red">*</span></label>
                                    <input id="titleYomi" v-model="titleYomi" type="text" placeholder="ガッキョクメイ"
                                        class="w3-input w3-border w3-round" />
                                        <hr>
                                    <label for="lyrics"><b>歌詞</b><span class="w3-text-red">*</span></label>

                                    <textarea id="lyrics" v-model="lyrics" type="text" maxlength="1000" placeholder="歌詞を入力"
                                        class="w3-input w3-border w3-round">{{ lyrics }}</textarea>

                                    <p :class='lyrics.length == 1000 ? "w3-pale-red" : ""'><span>{{ lyrics.length
                                            }}/1000文字</span>
                                    </p>
                                    <p>または.txtファイルをアップロード</p><input type="file" accept=".txt" class="w3-input" @change="handleLyricsFile"/>



                                    <label for="audioFileLink"><b>音源</b><span class="w3-text-red">*</span></label>
                                    <input id="audioFileLink" v-model="audioFileLink" type="url" placeholder="音源リンク https://" class="w3-input  w3-border w3-round" />
                                    <p>またはMP3ファイルをアップロード</p>
                                    <input type="file" accept=".mp3" class="w3-input" @change="handleAudioUpload"/>
                                    <audio :src="audioSrc" controls v-if="audioSrc"></audio>
                                    
                                    <label><b>翻訳先言語(複数可)</b><span class="w3-text-red">*</span></label>

                                    <div>
                                        <div v-for="language in languages">
                                            <input class="w3-check" type="checkbox" :id="language.id" :value="language"
                                                v-model="selectedLanguages" />
                                            <label :for="language.id">{{ language.value }}</label><br>
                                        </div>
                                    </div>
                                </section>

                                <hr />
                                <section>
                                    <h6 class="w3-text-gray">オプション</h6>

                                    <input class="w3-check" type="checkbox" v-model="needSRT">
                                    <label><b>srtファイル作成(+¥{{ srtFee }})</b></label><br>
                                    <!-- <input class="w3-check" type="checkbox" v-model="haveSRT">
                                    <label><b>既にsrtファイルを持っています</b></label>
                                    <div v-if="haveSRT">
                                        <input type="file" class="w3-input"/>
                                    </div> -->
                                </section>

                                <hr />
                                <label><b>金額</b></label>
                                <p>¥{{ languageFee * selectedLanguages.length + (needSRT ? srtFee : 0) }}</p>

                            </form>
                            <p v-if="isErrMsg" class="w3-red">{{ errMsg }}</p>
                            <button @click="cancelEditing"
                                class="w3-button w3-round-large w3-light-gray w3-display-bottomlight">キャンセル</button>
                            <button @click="finishEditingSong(song.id)"
                                class="w3-button w3-round-large w3-lime w3-display-bottomright">保存</button>
                        </div>
                    </td>
                </template>
            </tr>
        </tbody>
        </table>
    </div>
    <div>
        <h2>合計: ¥{{ songs.map((song) => song.price).reduce((a, b) => a + b, 0) }}</h2>
        <h3>納品予定日: {{ formattedDate }}</h3>
    </div>
    <div v-if="isLimitModal" class="w3-modal" :style="{ display: isLimitModal ? 'block' : 'none' }">
        <div class="w3-modal-content w3-round-large w3-animate-top w3-center">
            <header>
                <h2 class="w3-lime w3-round-large">曲数の上限です</h2>
            </header>
            <p class="w3-padding-16">一度に注文できるのは20曲までとなります。20曲を超えるご依頼については、下記フォームよりお問い合わせくださいませ。<br><a
                    href="https://docs.google.com/forms/d/e/1FAIpQLScy7Bg7KTxaXv04KJoLBAsSVbkba4f183kITNadGhWZTJjmsw/viewform"
                    target="_blank">お問い合わせフォームへ</a></p>
            <footer>
                <button @click="closeLimitModal" class="w3-btn w3-round-large w3-lime">OK</button>
            </footer>
        </div>
    </div>
    <div v-if="isDeleteModal" class="w3-modal" :style="{ display: isDeleteModal ? 'block' : 'none' }">
        <div class="w3-modal-content w3-round-large w3-animate-top w3-center">
            <p class="w3-padding-16">『{{ selectedSong.title }}』を削除しますか？</p>
            <footer>
                <button @click="closeDeleteModal" class="w3-btn w3-round-large w3-light-gray">キャンセル</button>
                <button @click="deleteSong" class="w3-btn w3-round-large w3-red">削除</button>
            </footer>

        </div>

    </div>
</template>