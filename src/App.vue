<template>
  <div class="App" :class="currentTheme">
    <div class="card studentNames" style="padding: 0;">
      <div class="class-wrapper">
        <button class="class-btn" :class="currentClass == 0 ? 'active-class' : ''" @click="setActive(0)" placeholder="Class 1">1</button>
        <button class="class-btn" :class="currentClass == 1 ? 'active-class' : ''" @click="setActive(1)" placeholder="Class 2">2</button>
        <button class="class-btn" :class="currentClass == 2 ? 'active-class' : ''" @click="setActive(2)" placeholder="Class 3">3</button>
        <button class="class-btn" :class="currentClass == 3 ? 'active-class' : ''" @click="setActive(3)" placeholder="Class 4">4</button>
        <button class="class-btn" :class="currentClass == 4 ? 'active-class' : ''" @click="setActive(4)" placeholder="Class 5">5</button>
        <button class="class-btn" :class="currentClass == 5 ? 'active-class' : ''" @click="setActive(5)" placeholder="Class 6">6</button>
        <button class="class-btn" :class="currentClass == 6 ? 'active-class' : ''" @click="setActive(6)" placeholder="Class 7">7</button>
      </div>
      <div class="student-control-header">Students</div>
      <div class="student-control">
        <button class="small-btn m-2" @click="addTempStudent">Edit</button>
        <button class="small-btn m-2" @click="clearStudents">Clear</button>
      </div>
      <hr>
      <div class="student-container">
        <div v-if="!addingStudent" class="student-wrapper-wrapper">
          <div class="student-wrapper" v-for="(item, i) in students" :key="i" v-on:mouseenter="nameHovered(i)" v-on:mouseleave="nameUnhovered(i)">
            <div class="student">{{item.name}}</div>
            <div v-if="item.btnShowing" class="delete-btn" @click="removeStudent(i)">&#10005;</div>
          </div>
        </div>
        <div class="temp-student-container" v-if="addingStudent">
          <textarea class="tempname-input" v-model="tempStudentName" placeholder="Enter students" />
          <div class="input-control">
            <button class="small-btn m-2" style="margin-left: 0;" @click="addStudent">Add</button>
            <button class="small-btn m-2" style="margin-right: 0;" @click="cancelAddStudent">Cancel</button>
          </div>
        </div>
      </div>
    </div>
    <div class="content">
      <div class="card header">
        <div><button class="small-btn" @click="generateRandomName">Generate Name</button></div>
        <div class="mode-toggle-wrapper" @click="switchTheme">
          <div class="mode-toggle" ref="themeToggle"></div>
        </div>
      </div>
      <main>
        {{randomName}}
      </main>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data(){
    return{
      students: [],
      tempStudentName: '',
      randomName: '',
      addingStudent: false,
      calledNames: [],
      currentClass: -1,
      currentTheme: ''
    }
  },
  methods: {
    addTempStudent(){
      this.addingStudent = true;
      let tempTempNames = [];
      this.students.forEach(item => {
        tempTempNames.push(item.name);
      });
      tempTempNames = tempTempNames.join('\n');
      this.tempStudentName = tempTempNames;
    },
    addStudent(){
      if(this.tempStudentName != ''){
        let nameList = this.tempStudentName.split('\n');
        this.students = [];
        let tempStudents = [];
        nameList.forEach(item => {
          tempStudents.push(item);
        });
        tempStudents.forEach((item, index) => {
          if(nameList[index] == item && index == nameList.indexOf(item)){
            this.students.push({name: item, btnShowing: false});
          }
        });
      }else{
        this.students = [];
      }
      this.tempStudentName = '';
      this.addingStudent = false;
      let storedData = JSON.parse(localStorage.getItem('students'));
      if(storedData == null){
        storedData = [];
      }
      storedData[this.currentClass] = this.students;
      localStorage.setItem('students', JSON.stringify(storedData));
    },
    generateRandomName(){
      if(this.students.length >= 4){
        //subtract 1 from times left of calledNames list
        for(let i = 0; i < this.calledNames.length; i++){
          this.calledNames[i].timesLeft--;
        }
        //filter through the names and keep ones with >0 value (.filter doesn't work so i do this)
        let filteredNames = [];
        this.calledNames.forEach(item => {
          if(item.timesLeft > 0){
            filteredNames.push(item);
          }
        });
        this.calledNames = filteredNames;
        //finding valid random name
        let calledStudent = this.students[Math.floor(Math.random()*this.students.length)].name;
        let namesList = [];
        this.calledNames.forEach(item => {
          namesList.push(item.name);
        });
        while(namesList.includes(calledStudent)){
          calledStudent = this.students[Math.floor(Math.random()*this.students.length)].name;
        }
        //push called student to called names list with correct timesleft
        if(this.students.length == 1){
          this.calledNames.push({name: calledStudent, timesLeft: 0});
        }else if(this.students.length < 5){
          this.calledNames.push({name: calledStudent, timesLeft: Math.floor(this.students.length/2)});
        }else{
          this.calledNames.push({name: calledStudent, timesLeft: 5});
        }
        this.randomName = calledStudent
      }else{
        alert('Class must have at least 4 students');
      }
    },
    removeStudent(i){
      let newArr = [];
      this.students.forEach((item, index) => {
        if(i != index){
          newArr.push(item);
        }
      });
      this.students = newArr;
      let storedData = JSON.parse(localStorage.getItem('students'));
      if(storedData == null){
        storedData = [];
      }
      storedData[this.currentClass] = newArr;
      localStorage.setItem('students', JSON.stringify(storedData));
    },
    nameHovered(i){
      this.students[i].btnShowing = true;
    },
    nameUnhovered(i){
      this.students[i].btnShowing = false;
    },
    switchTheme(){
      let preferedStyle = JSON.parse(localStorage.getItem('theme'));
      if(preferedStyle == 'light-theme'){
        localStorage.setItem('theme', JSON.stringify('dark-theme'));
        this.currentTheme = 'dark-theme';
        this.$refs.themeToggle.classList.remove('mode-toggle-dark');
      }else if(preferedStyle == 'dark-theme'){
        localStorage.setItem('theme', JSON.stringify('light-theme'));
        this.currentTheme = 'light-theme';
        this.$refs.themeToggle.classList.add('mode-toggle-dark');
      }
    },
    cancelAddStudent(){
      this.addingStudent = false;
    },
    setActive(num){
      this.currentClass = num;
      let savedData = JSON.parse(localStorage.getItem('students'));
      if(savedData[this.currentClass] != null){
        this.students = savedData[this.currentClass];
      }else{
        this.students = [];
      }
    },
    clearStudents(){
      let savedData = JSON.parse(localStorage.getItem('students'));
      this.students = [];
      savedData[this.currentClass] = [];
      localStorage.setItem('students', JSON.stringify(savedData));
    }
  },
  async created(){
    // localStorage.setItem('students', JSON.stringify([]));
    let currentClass = JSON.parse(localStorage.getItem('currentClass'));
    if(currentClass != null){
      this.currentClass = currentClass;
    }else{
      this.currentClass = 0;
    }

    let jsonData = JSON.parse(localStorage.getItem('students'));
    if(jsonData != null && jsonData[this.currentClass] != null){
      this.students = jsonData[this.currentClass];
    }

    await this.$refs;
    let preferedStyle = JSON.parse(localStorage.getItem('theme'));
    if(preferedStyle != null){
      this.currentTheme = preferedStyle;
      if(preferedStyle == 'light-theme'){
        this.$refs.themeToggle.classList.add('mode-toggle-dark');
      }
    }else{
      localStorage.setItem('theme', JSON.stringify('light-theme'));
      this.currentTheme = 'light-theme';
      if(preferedStyle == 'light-theme'){
        this.$refs.themeToggle.classList.add('mode-toggle-dark');
      }
    }
  },
}
</script>

<style>
:root{
  --green: #2ecc71;
  --classBtn: #efefef;
}
.dark-theme{
  --bgColor: #2c3e50;
  --cardColor: #34495e;
  --textColor: #ffffff;
  --bgOpposite: #e9e9e9;
}
.light-theme{
  --bgColor: #e9e9e9;
  --cardColor: #ffffff;
  --textColor: #000000;
  --bgOpposite: #2c3e50;
}
.App{
  height: 100vh;
  display: flex;
  flex-direction: row;
  padding: 15px;
  background: var(--bgColor);
  transition: 0.15s;
}
.studentNames{
  height: 100%;
  max-height: 100%;
  border: none;
  border-radius: 15px;
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.25);
  width: 250px;
  min-width: 250px;
  max-width: 250px;
  display: flex;
  flex-direction: column;
  overflow-y: scroll;
  overflow-x: unset;
}
.studentNames::-webkit-scrollbar{
  display: none;
}
button{
  white-space: nowrap;
}
.card{
  border: none;
  border-radius: 15px;
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.25);
  padding: 15px;
  background: var(--cardColor);
  transition: 0.15s;
}
.small-btn{
  padding: 6px 18px;
  border: none;
  border-radius: 8px;
  outline: none;
  transition: 0.15s;
  cursor: pointer;
  background: var(--green);
  color: white;
}
.small-btn:hover{
  background-color: #6ee09e;
}
.student-control{
  display: flex;
  flex-direction: row;
  justify-content: center;
  padding-top: 0;
  padding-bottom: 0;
}
input{
  border: 1px solid black;
  border-radius: 4px;
  outline: none;
}
.student-control-header{
  width: 100%;
  padding: 8px;
  padding-bottom: 0;
  padding-top: 0;
  display: flex;
  justify-content: center;
  font-weight: bold;
  font-size: 22px;
  color: var(--textColor);
  transition: 0.15s;
}
hr{
  width: calc(100% - 40px) !important;
  height: 0px !important;
  border: 1px solid var(--textColor);
  margin-left: 20px;
  margin-right: 20px;
  margin-top: 4px;
  margin-bottom: 0;
  transition: 0.15s;
}
.student-container{
  padding: 15px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  max-width: 250px;
  height: calc(100% - 120px);
}
.student{
  padding: 6px;
  border: 1px solid #ccc;
  border-radius: 8px;
  color: var(--textColor);
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  user-select: none;
  -moz-window-dragging: none;
  cursor: default;
  transition: 0.15s;
}
.m-2{
  margin: 4px;
}
.tempname-input{
  width: 100%;
  padding: 4px;
  height: 1.6rem;
}
.center{
  display: flex;
  justify-content: center;
  align-items: center;
}
.temp-student-container{
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
}
.temp-student-container > div{
  display: flex;
  flex-direction: row;
}
.temp-student-container > div > button{
  width: 100%;
}
.temp-student-container > textarea{
  height: 60%;
  width: 100%;
  height: 100%;
  outline: none
}
.content{
  height: 100%;
  width: 100%;
  display: flex;
  flex-direction: column;
  padding: 15px;
  padding-top: 0;
  padding-right: 0;
}
.header{
  width: 100%;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}
main{
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  width: 100%;
  max-width: 100%;
  word-break: break-all;
  text-align: center;
  font-size: 50px;
  font-weight: bold;
  color: #2ecc71;
}
.student-wrapper{
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  max-width: 250px;
  margin-bottom: 8px;
}
.student-wrapper > button{
  height: 100%;
}
.student-wrapper > input{
  height: 100%;
}
.delete-btn{
  margin-right: 8px;
  margin-left: 14px;
  cursor: pointer;
  color: var(--textColor);
  transition: 0.15s;
  user-select: none;
  -webkit-user-drag: none;
}
.mode-toggle-wrapper{
  width: 44px;
  height: 26px;
  padding: 2px;
  background: var(--bgOpposite);
  border-radius: 100px;
  cursor: pointer;
}
.mode-toggle{
  height: 22px;
  width: 22px;
  border: none;
  border-radius: 100px;
  background: var(--green);
  transition: 0.15s;
  transform: translateX(0px);
}
.mode-toggle-dark{
  transform: translateX(18px);
}
.mode-toggle-dark:active{
  transform: translateX(14px);
}
.mode-toggle:active{
  width: 26px !important;
}
.student-wrapper-wrapper{
  width: 100%;
}
.input-control{
  width: 100%;
}
.class-btn{
  padding: 6px;
  border: 2px solid #efefef;
  border-radius: 8px;
  outline: none;
  transition: 0.15s;
  cursor: pointer;
  margin: 4px;
  background-color: var(--classBtn);
  width: 100%;
}
.class-btn:hover{
  background-color: #ccc;
  border-color: #ccc;
}
.active-class{
  border-color: var(--green) !important;
}
.class-wrapper{
  padding-bottom: 12px;
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  justify-content: center;
  align-items: center;
  padding: 4px;
}
</style>