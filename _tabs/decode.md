---
title: Decode Game
order: 4
math: true
---


This is a "game" loosely inspired by the decoding problem from error correction. Note that the graph below is randomly generated and does not correspond to any sort of good code. Click the bluish nodes to turn off the greenish nodes. Refresh to generate a new graph. For now, this is best viewed on a desktop.
<div id="game-container"></div>


<script src="/assets/js/phaser.min.js"></script>
<script>

  let winWidth = 800;
  let winHeight = 600;
  let config = {
    type: Phaser.AUTO,
    width: winWidth,
    height: winHeight,
    backgroundColor: '#1B1B1E',
    parent: 'game-container',
    scene: {
        create: create,
    }
  };
  
  
  let game = new Phaser.Game(config);
  let text;
  let textBox;
  
  let numBits = 16;
  let numChecks = 8;
  let bitsToChecks = {}
  
  
  
  const radius = 20;
  const border = 25;
  const innerBorder = 25;
  
  
  const bitOnColor = 0x8AB4F8;
  const bitOffColor = 0x53688B;
  const checkOnColor = 0x7DDE92;
  const checkOffColor = 0x4C7D58;
  const lineColor = 0xF8FFE5;
  
  const bits = [];
  const checks = [];
  
  const bitVals = [];
  const checkVals = [];
  
  const bitIndices = []
  for (let i=0; i<numBits; i++) {
    bitIndices.push(i)
  }
  
  let clickedBits = 0;
  let totalClicks = 0;
  
  
  function create () {

    generateBitsToChecks();
    clickedBits = 0;
    totalClicks = 0;
    let numErrors = getRandomInt(6, 3);
    errors = getRandomSubarray(bitIndices, numErrors)
    
  
    // draw lines and create bit nodes
    for (let b=0; b < numBits; b++) 
    {
      let chks = bitsToChecks[b];
      let pos1 = getBitXY(b,numBits,winWidth,winHeight,border);
      for (let i=0; i < chks.length; i++){
        let c = chks[i];
        let pos2 = getCheckXY(c, numChecks, winWidth, winHeight, innerBorder);
        let lin = this.add.line(0, 0, pos1[0], pos1[1], pos2[0], pos2[1],lineColor);
        lin.setOrigin(0,0);
      }
      let bit = this.add.circle(pos1[0],pos1[1],radius,bitOffColor);
      bit = bit.setName(b.toString());
      bit = bit.setInteractive();
      bit = bit.on('pointerdown', () => toggleBit(b));
      
      bits.push(bit);
      bitVals.push(false);
    }
  
    // create check nodes
    for (let c=0; c < numChecks; c++) {
      let pos = getCheckXY(c, numChecks, winWidth, winHeight, innerBorder);
      checks.push(this.add.circle(pos[0],pos[1],radius,checkOffColor));
      checkVals.push(false)
    }
  
    for (let i=0; i < numErrors; i++) {
      toggleBit(errors[i], active=false)
    }
    
    textBox = this.add.graphics();

    textBox.fillStyle(0x1B1B1E, 1);
    textBox.fillRect(0, 0, 800, 600);
    textBox.setVisible(false);

    text = this.add.text(400, 300, ' ', { fontFamily: 'Arial', fontSize: 32, color: '#8AB4F8' });
    text.setOrigin(0.5);
    text.setVisible(false);
  
  }
  

  
  function getBitXY(idx, maxIdx, winW, winH, bord) {
    let topLen = Math.floor((maxIdx + 1) / 2);
    if ( idx < topLen ) {
      let delta = (winW - 2*bord) / topLen;
      x = bord + delta / 2 + idx * delta;
      y = bord;
    }
    else {
      let delta = (winW - 2*bord) / (maxIdx - topLen);
      x = bord + delta / 2 + (idx - topLen) * delta;
      y = winH - bord;
    }
    return [x, y];
  }
  
  function getCheckXY(idx, maxIdx, winW, winH, bord) {
    let delta = (winW - 2 * bord) / maxIdx;
    x = bord + delta / 2 + idx * delta;
    y = winH / 2;
    return [x,y];
  }
  
  function toggleCheck(checkIdx) {
    checkVal = checkVals[checkIdx];
    check = checks[checkIdx];
    if (checkVal) {
      checkVals[checkIdx] = false;
      check.fillColor = checkOffColor;
    }
    else {
      checkVals[checkIdx] = true;
      check.fillColor = checkOnColor;
    }
  }
  
  function toggleBit(bitIdx, active=true) {
    bitChecks = bitsToChecks[bitIdx];
    bit = bits[bitIdx]
    let i = 0;
    while (i < bitChecks.length) {
      checkIdx = bitChecks[i];
      toggleCheck(checkIdx);
      i++;
    }
    if (bitVals[bitIdx] && active) {
      bitVals[bitIdx] = false;
      bit.fillColor = bitOffColor;
      clickedBits = clickedBits - 1;
      totalClicks++;
    }
    else if (active) {
      bitVals[bitIdx] = true;
      bit.fillColor = bitOnColor;
      clickedBits = clickedBits + 1;
      totalClicks++;
    }
  
    if (active) {
      completion();
    }
  }
  
  function completion() {
    let complete = true;
    for(let i=0; i < numChecks; i++) {
      if (checkVals[i]) {
        complete = false;
      }
    }
    if (complete) {
      let message = `You succeeded with ${clickedBits} clicked nodes in ${totalClicks} total clicks.`;
      text.setText(message);
      textBox.setVisible(true);
      text.setVisible(true);
      setTimeout(function() {
        startGame();
      }, 4000);
    }
  }
  
  function generateBitsToChecks() {
    let edgeFraction = 0.3;
    let numEdges = Math.floor(numBits * numChecks * edgeFraction);
  
    // generate a list of check indices, ensuring each check is preset
    let checkList = [];
    for (let cIdx = 0; cIdx < numChecks; cIdx++) {
      checkList.push(cIdx);
      checkList.push(cIdx);
    }
    
    // populate checkList with random check indices
    for (let i=0; i < numEdges - 2 * numChecks; i++) {
      checkList.push(getRandomInt(numChecks));
    }
  
    shuffleArray(checkList);
  
  
    // initialize bitsToChecks with 2 checks per bit
    for (let bIdx =0; bIdx < numBits; bIdx++) {
      bitsToChecks[bIdx] = checkList.slice(2*bIdx, 2*bIdx+2);
    }
  
    // assign remaining checks to random bits
    let cIdx = 2 * numBits;
    let bIdx = 0;
    while (cIdx < checkList.length) {
      bIdx = getRandomInt(numBits);
      bitsToChecks[bIdx].push(checkList[cIdx]);
      cIdx++;
    }
    
    // remove duplicate checks
    for (let bIdx =0; bIdx < numBits; bIdx++) {
      bitsToChecks[bIdx] = Array.from(new Set(bitsToChecks[bIdx]));
    }
  }
  
  function startGame() {
    textBox.setVisible(false);
    text.setVisible(false);
    for (let bIdx=0; bIdx < numBits; bIdx++) {
      bitVals[bIdx] = false;
      let bit = bits[bIdx]
      bit.fillColor = bitOffColor;
    }
    let numErrors = getRandomInt(6, 3);
    errors = getRandomSubarray(bitIndices, numErrors)
    for (let i=0; i < numErrors; i++) {
      toggleBit(errors[i], active=false)
    }
    
    clickedBits = 0;
    totalClicks =0;
  }
  
  function shuffleArray(array) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      const temp = array[i];
      array[i] = array[j];
      array[j] = temp;
    }
  }
  
  function getRandomInt(upper, lower=0) {
    let diff = upper - lower;
    return lower + Math.floor(Math.random() * diff);
  }
  
  function getRandomSubarray(arr, size) {
    var shuffled = arr.slice(0), i = arr.length, temp, index;
    while (i--) {
      index = Math.floor((i + 1) * Math.random());
      temp = shuffled[index];
      shuffled[index] = shuffled[i];
      shuffled[i] = temp;
    }
    return shuffled.slice(0, size);
  }
  
  
  </script>
