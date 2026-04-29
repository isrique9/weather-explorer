<template>
<div id="frame"> 
  <div class="sun">
    <div class="sun__cloud sun__cloud--bg-1">
      <div class="sun__cloud__border">
        <div> </div>
        <div> </div>
        <div> </div>
        <div> </div>
      </div>
    </div>
    <div class="sun__cloud sun__cloud--bg-2">
      <div class="sun__cloud__border">
        <div> </div>
        <div> </div>
        <div> </div>
        <div> </div>
      </div>
    </div>
    <div class="sun__cloud sun__cloud--fg">
      <div class="sun__cloud__border">
        <div> </div>
        <div> </div>
        <div> </div>
        <div> </div>
      </div>
    </div>
    <div class="sun__shadow"> </div>
    <div class="sun__border">
      <div class="sun__inner">
        <div class="sun__body">
          <div class="sun__shine"></div>
          <div class="sun__face">
            <div class="sun__face-eye sun__face-eye--left"></div>
            <div class="sun__face-eye sun__face-eye--right"> </div>
            <div class="sun__face-mouth"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
</template>

<style scoped>
html,
body {
  padding: 0;
  margin: 0;
  width: 100%;
  height: 100%;
  background: #3498db;
}

#frame {
  display: flex;
  width: 100%;
  height: 100%;
  align-items: center;
  justify-content: center;
}

:root {
  --shadow: #33333329;
  --sun-yellow: #fdff7b;
  --sun-shadow: #fff14d;
  --sun-border: #ffca00;
  --sun-face: #333;
  --sun-mouth: #ff506f;
}

.sun {
  position: relative;
  width: 500px;
  height: 500px;
}

.sun__cloud {
  position: absolute;
  width: 50px;
  height: 10px;
}

.sun__cloud__border {
  position: relative;
  margin: 30px 0 0 0;
  width: 100%;
  height: 100%;
  background: #f1f2f6;
}

.sun__cloud__border div {
  position: absolute;
  border-radius: 50%;
  width: 12px;
  height: 12px;
  left: -5px;
  bottom: 0;
  box-shadow: inset -2px -3px 0 0 #f1f2f6;
  background: #fff;
  z-index: 10;
}

.sun__cloud__border div:nth-child(1) {
  transform: scale(1.5);
  left: 10%;
  bottom: 30%;
  z-index: 9;
}

.sun__cloud__border div:nth-child(2) {
  transform: scale(2.5);
  left: 50%;
  bottom: 90%;
  z-index: 8;
}

.sun__cloud__border div:nth-child(3) {
  transform: scale(1.1);
  left: 84%;
  bottom: 4.5%;
  z-index: 7;
}

.sun__cloud--fg {
  top: 60%;
  left: 15%;
  z-index: 15;
  animation: cloud-left 15s linear infinite;
}

.sun__cloud--fg > div {
  transform: rotateY(180deg) scale(1.5);
}

.sun__cloud--bg-1 {
  top: 35%;
  right: 5%;
  z-index: 1;
  animation: cloud-right 12.5s linear infinite;
}

.sun__cloud--bg-1 > div {
  transform: scale(2);
}

.sun__cloud--bg-2 {
  top: 20%;
  left: 20%;
  z-index: 1;
  animation: cloud-right-far 10s linear infinite;
}

.sun__cloud--bg-2 > div {
  transform: rotateY(180deg) scale(1.2);
}

.sun__shadow {
  position: absolute;
  width: 40%;
  height: 20px;
  top: 100%;
  left: 30%;
  border-radius: 50%;
  background: var(--shadow);
  animation: shadow 5s ease-out infinite;
}

.sun__border {
  position: relative;
  width: 100%;
  height: 100%;
  z-index: 10;
  animation: float 5s ease-out infinite;
}

.sun__border * {
  position: absolute;
}

.sun__outer {
  width: 227.5%;
  height: 227.5%;
  top: -63.75%;
  left: -63.75%;
  animation: rotate 80s linear infinite;
}

.sun__rays {
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  top: 46.5%;
  left: 46.5%;
  width: 7%;
  height: 7%;
}

.sun__rays div {
  width: 60%;
  height: 30%;
  background: var(--sun-border);
  border-radius: 20px;
}

.sun__rays:nth-child(1) { transform: rotate(0deg) translate(227.5%); }
.sun__rays:nth-child(2) { transform: rotate(45deg) translate(227.5%); }
.sun__rays:nth-child(3) { transform: rotate(90deg) translate(227.5%); }
.sun__rays:nth-child(4) { transform: rotate(135deg) translate(227.5%); }
.sun__rays:nth-child(5) { transform: rotate(180deg) translate(227.5%); }
.sun__rays:nth-child(6) { transform: rotate(225deg) translate(227.5%); }
.sun__rays:nth-child(7) { transform: rotate(270deg) translate(227.5%); }
.sun__rays:nth-child(8) { transform: rotate(315deg) translate(227.5%); }

.sun__inner {
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.sun__body {
  width: 50%;
  height: 50%;
  top: 23%;
  left: 23%;
  border-radius: 50%;
  border: 15px solid var(--sun-border);
}

.sun__body:before {
  content: "";
  position: absolute;
  top: 0px;
  left: 0px;
  width: 100%;
  height: 100%;
  border-radius: 100%;
  background: var(--sun-yellow);
  box-shadow: inset 15px 0 0 var(--sun-border), inset 25px -50px 0 var(--sun-shadow);
}

.sun__shine {
  top: -5%;
  left: -5%;
  width: 110%;
  height: 110%;
  border-radius: 100%;
  box-shadow: 0 0 0 10px #ffff0014, 0 0 0 15px #ffff0014, 0 0 0 30px #ffff0014;
  animation: shine 5s ease-in infinite;
}

.sun__face {
  width: 100%;
  height: 100%;
}

.sun__face-eye {
  top: 50%;
  width: 15px;
  height: 15px;
  border-radius: 50%;
  background: var(--sun-face);
  animation: blink 2s linear infinite;
}

.sun__face-eye--left {
  right: 32%;
  position: absolute;
}

.sun__face-eye--right {
  right: 16%;
  position: absolute;
}

.sun__face-mouth {
  width: 25px;
  height: 17px;
  bottom: 36.2%;
  right: 22%;
  position: absolute;
  box-sizing: border-box;
  border-radius: 4px 4px 25px 25px;
  background: var(--sun-face);
  overflow: hidden;
}

.sun__face-mouth:before {
  content: "";
  position: absolute;
  top: 40%;
  left: -15%;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: var(--sun-mouth);
}

@keyframes float {
  0% { transform: translateY(0px); }
  50% { transform: translateY(20px); }
  100% { transform: translateY(0px); }
}

@keyframes cloud-left {
  0% { transform: translateX(0px); }
  50% { transform: translateX(20px); }
  100% { transform: translateX(0px); }
}

@keyframes cloud-right {
  0% { transform: translateX(0px); }
  50% { transform: translateX(-20px); }
  100% { transform: translateX(0px); }
}

@keyframes cloud-right-far {
  0% { transform: translateX(0px); }
  50% { transform: translateX(-40px); }
  100% { transform: translateX(0px); }
}

@keyframes shadow {
  0% { transform: scaleX(1); }
  50% { transform: scaleX(0.775); }
  100% { transform: scaleX(1); }
}

@keyframes rotate {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

@keyframes shine {
  0% { transform: scale(1); }
  50% { transform: scale(0.965); }
  100% { transform: scale(1); }
}

@keyframes blink {
  0% { transform: scaleY(1); }
  5% { transform: scaleY(0.3); }
  10% { transform: scaleY(1); }
  100% { transform: scaleY(1); }
}
</style>