<template>
  <section>
    <h1> {{title}} </h1>
    <h2 v-if="isQuizPlaying"> [ジャンル]：{{genre}} </h2>
    <h2 v-if="isQuizPlaying"> [難易度]：{{difficulty}} </h2>
    <hr>
    <p> {{description}} </p>
    <hr>
    <button @click="onOk" v-if="isStart">開始</button>
    <button v-for="answer in answers" v-if="isQuizPlaying" @click="checkAnswer" :value="answer"> {{ answer }} </button>
    <button @click="restartQuiz" v-if="isFinish">ホームに戻る</button>
  </section>
</template>

<script>
export default {
    data: () => ({
        title: '',
        description: '',
        genre: '',
        difficulty: '',
        Data: [],
        isQuizPlaying: false,
        index: 0,
        answers: [],
        correctCount : 0,
        nowQuizData: [],
        isStart: true,
        isFinish: false
    }),
    created () {
        this.title = 'クイズへようこそ';
        this.description = 'クイズを開始する場合は「開始」ボタンを押してください';
    },
    methods: {
        getQuizData() {
            return fetch('https://opentdb.com/api.php?amount=10&type=multiple')
                .then((response) => {
                    return response.json();
                })
                .then(res => {
                    this.Data = res;
                })
        },
        onOk() {
            this.title = '取得中';
            this.description = '少々お待ちください';
            this.isStart = false;
            this.getQuizData().then(() => {
                this.isQuizPlaying = true;
                this.showQuiz();
                })
        },
        htmlCodesDecoode(text) {
            text = text.replace(/\&amp\;/g,"&");
            text = text.replace(/\&quot\;/g,'"');
            text = text.replace(/\&#039\;/g,"'");
            text = text.replace(/\&lt\;/g,"<");
            text = text.replace(/\&gt\;/g,">");
            return text ;
        },
        shuffleArray(array) {
            for (let i = array.length - 1; i >= 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        },
        showQuiz() {
            this.nowQuizData = this.Data.results[this.index];
            this.answers = this.answers.concat(this.nowQuizData.incorrect_answers);
            this.answers.push(this.nowQuizData.correct_answer);
            this.answers = this.shuffleArray(this.answers);
            this.title = '問題' + (this.index+1);
            this.genre = this.htmlCodesDecoode(this.nowQuizData.category);
            this.difficulty = this.nowQuizData.difficulty;
            this.description = this.htmlCodesDecoode(this.nowQuizData.question);
            this.index++;
        },
        checkAnswer(event){
            if(event.target.value === this.nowQuizData.correct_answer){
                this.correctCount++;
            }
            this.answers = [];
            this.nextQuiz();

        },
        nextQuiz(){
            if(this.index !== 10){
                this.showQuiz();
            } else {
                this.isQuizPlaying = false;
                this.title = 'あなたの正解数は' + this.correctCount + 'です。';
                this.description = '再挑戦したい場合は以下をクリック';
                this.isFinish = true;
            }
        },
        restartQuiz(){
            this.isStart = true;
            this.correctCount = 0;
            this.index = 0;
            this.title = 'クイズへようこそ';
            this.description = 'クイズを開始する場合は「開始」ボタンを押してください';
            this.isFinish = false;
        }
    }
}
</script>

<style scoped>
button {
    display: block;
    margin-bottom: 10px;
}
</style>