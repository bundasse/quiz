<template>
  <div class="w-full flex items-center justify-center flex-wrap h-full">
    <div class="mx-auto w-10/12 lg:w-7/12 flex items-center flex-wrap">
      <div class="h-5 bg-gray-300 basis-full relative rounded-lg">
        <div class="absolute rounded-lg h-5 bg-blue-500 transition-all duration-500" :style="{width: progress+'%'}"></div>
        <!-- 만약 현재 문제를 진행중이라면 x/전체 문제수 로 표시되고, 그게 아니라면 종료되었으므로 다른 메세지를 보여준다. -->
        <p v-if="current != Number(selectCount)" class="absolute right-2 -top-5 text-xs">{{ current+1 }} / {{ selectCount }}</p>
        <p v-else class="absolute right-2 -top-5 text-xs">종료</p>
        <p class="absolute right-2 top-0.5 text-xs">{{ progress+"%" }}</p>
      </div>
      <h3 class="font-bold basis-full text-center text-indigo-500 sm:text-2xl lg:text-3xl text-xl mt-10 bg-white rounded-lg p-5">{{ userName }}<span class="text-black">님 반갑습니다.</span></h3>
      <ul class="flex mt-3 gap-x-4 items-center">
        <li>현재설정</li>
        <li class="px-2 py-1 rounded bg-blue-500 text-white">{{ selectRandom === '0' ? '기본': '랜덤' }}</li>
        <li class="px-2 py-1 rounded bg-rose-500 text-white">문제유형 : {{ selectType}}</li>
        <li class="px-2 py-1 rounded bg-green-500 text-white">문제수: {{ selectCount}}</li>
      </ul>
      <!-- 문제영역 -->
      <div class="basis-full bg-white rounded-lg my-10 p-10" v-if="current < Number(selectCount)">
        {{ current+1 }}번 문제
        <!-- {{ dataList.QuizList[current] }} -->
        <p class="text-base sm:text-xl">{{ selectQuestion[current].question }}</p>
        <ul class="flex flex-wrap justify-between mt-5">
          <li v-for="(e,index) in randomView[current]" :key="index" class="cursor-pointer font-bold p-2.5 lg:p-5 text-sm basis-full lg:basis-[49%] rounded-lg mb-5 border hover:bg-blue-500 transition-all duration-200 hover:text-white flex" @click="current++; SelectValue(e); hintUse=false;">
            <span>{{ index + 1 }} 번 : </span>
            <span class="break-all text-center basis-10/12">{{ e[1] }}</span>
          </li>
        </ul>
        <div class="flex justify-between items-start flex-wrap">
          <button class="btn_primary p-1 rounded bg-green-400 hover:bg-green-600 hover:text-white focus:ring-green-400 basis-4/12 sm:basis-2/12 text-sm" @click="useHint()"> 힌트 : {{ hint }}개</button>
            <p v-if="hintUse" class="basis-8/12 sm:basis-10/12 text-right">{{ selectQuestion[current].hint }}</p>
            <p v-else-if="hint<1" class="basis-8/12 sm:basis-10/12 text-right text-rose-500">힌트를 모두 사용하셨습니다.</p>
          </div>
          <div class="flex justify-end flex-wrap gap-x-2">
            <router-link to="/" class="cursor-pointer font-bold px-3 py-1 text-sm rounded-lg mt-5 border hover:border-green-400 transition-all duration-100 hover:text-green-400">처음으로 돌아가기</router-link>
            <button @click="current++; SelectValue('모름'); hintUse=false" class="cursor-pointer font-bold px-3 py-1 text-sm rounded-lg mt-5 border hover:border-rose-500 transition-all duration-100 hover:text-rose-500">이 문제 포기하기</button>
          </div>
      </div>
      <div class="basis-full bg-white rounded-lg my-10 p-10 text-center font-bold text-lg" v-else>
        <span class="text-blue-500">{{resultScore}}</span>개 맞았습니다! 당신의 점수는 <span class="text-blue-500">{{ resultScore100 }}</span>점 입니다.
        {{ userSelect }}
        <p class="mt-3">
          <router-link to="/" class="cursor-pointer font-bold px-3 py-1 text-sm rounded-lg mt-5 border hover:border-green-400 transition-all duration-100 hover:text-green-400">처음으로 돌아가기</router-link>
        </p>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import QuizList from "../assets/QuizList.json"
interface QuizType {
  id: number,
  question: string,
  answer: string,
  view:{
    number1: string;
    number2: string;
    number3: string;
    number4: string;
  },
  type: string,
  hint: string
}
export default defineComponent({
  name: "QuizView",
  data() {
    return {
      dataList: QuizList,
      current: 0,
      userSelect : [] as string[],
      //type assertion : 변수 as 변경할 타입 : vue 값  as 변경할타입
      /*
      as 키워드는 유니온타입같은 복잡한 타입을 하나의 정확한 타입으로 줄일 때 사용.
      단, 하나의 타입일 때 사용시 에러 발생.
      as타입은 타입실드 임시해제용
      1.왜 타입 에러가 나는지 모를때 임시로 해결할 때 사용
      2.내가 어떤 타입 데이터가 들어올지 확실히는 모를때 에러방지용으로 사용
      */
      hint:2,
      hintUse : false,
      userName : this.$route.query.userName,
      selectType : this.$route.query.selectType,
      selectCount : this.$route.query.selectCount,
      selectRandom : this.$route.query.selectRandom,
      MaxCount:0
    }
  },
  methods: {
    SelectValue(e :unknown){
      this.userSelect.push((e as string)[1]);
      console.log(this.userSelect)
    },
    useHint() :void{
      if (this.hintUse){return}
      if(this.hint < 1){return}
      this.hint--;
      // this.hintUse === false ? this.hintUse = true : this.hintUse = false;
      this.hintUse ||= true
      // 평가논리 계산법. 값이 false일 때 true로만 변경 가능. 그 외의 값은 변경되지 않는다. 만약 여러 조건문이 필요하다면 if문으로 사용.
    },
    questionCount(){
      this.MaxCount = this.dataList.QuizList.filter((e)=>{
        if (this.selectType !== '전체') {
          return e.type === this.selectType
        }else{
          return e.type
        }
      }).length
    }
  },
  computed:{
    progress() :number{
      return Math.floor((this.current / Number(this.selectCount))*100);
      /*
      progress라는 이름의 타입을 넘버로 설정.
      해당 값을 리턴하는 값은 나머지를 계산하지 않고 버림
      계산내용 : 현재문제번호 / 문제 개수 *100
       */
      // floor : 소수점 나머지 내림 / ceil : 소수점 올림 / round : 소수점 반올림.
      /*
      const num = 3.141592;
      const ceil = Math.ceil(num) = 4
      const floor = Math.floor(num) = 3
      const round = Math.round(num) = 3(반올림.)
      */
    },
    resultScore():number{
      return this.selectQuestion.filter((e,index)=>{
        return e.answer ===this.userSelect[index]
      }).length
    },
    resultScore100():number{
      return Math.round((this.resultScore / Number(this.selectCount))*100);
    },
    randomView() :Array<Array<[string,unknown]>>{
      return this.selectQuestion.map((e,index)=>{
        return Object.entries(this.selectQuestion[index].view).sort(()=> Math.random()-0.5)
      })
    },
    selectQuestion() :QuizType[]{
      return this.dataList.QuizList.filter((e)=>{
        if (this.selectType !== '전체') {
          return e.type === this.selectType
        }else{
          return e.type
        }
      })
    }
  },
  created() {
    if(Number(this.selectRandom) != 0){
      this.selectRandom = '1';
    }
    if (this.selectRandom === '1') {
      this.dataList.QuizList.sort(()=> Math.random()-0.5)
    }
    this.questionCount();
    if(Number(this.selectCount) > this.MaxCount){
      this.selectCount = this.MaxCount.toString();
    }
  },
})
</script>

<style scoped>

</style>