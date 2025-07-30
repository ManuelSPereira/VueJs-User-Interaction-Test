<script setup lang="ts">
import { type ComponentPublicInstance, computed, onMounted, ref, registerRuntimeCompiler, triggerRef, useTemplateRef } from 'vue';




type CardInternal = {

  text: string;

  img?: HTMLImageElement;

  classList: string[];

  style: {key: string, value: string} [];
  

}

type GridCellInternal = {

  style: {key: string, value: string} [];

  card?: CardInternal | null;

}

type SelectedCardType = {

  card: CardInternal | null;
  refCard: HTMLElement | null;

}


onMounted(() => {

  document.body.addEventListener('mousemove', bodyMouseMoveEvHandler);
  document.body.addEventListener('mouseup', bodyMouseUpEvHandler);


});

const gridRef = useTemplateRef('grid');

const gridCellsPos = ref<GridCellInternal[][]>(initializeEmptyGridArray()); 
const gridCellsRef = ref<HTMLElement[][]>(initializeGridArrayRef());

const customCardInput = document.createElement('input');
customCardInput.type ='file';
customCardInput.accept = 'image/*';
customCardInput.addEventListener('change', (event : Event) => onIncomingCard(event));

const canAddCard = computed(() => {

  let can = false;

  gridCellsPos.value.forEach((xArray, yIndex) => {

    xArray.forEach((cell, xIndex) => {

      if(!cell.card) can = true;
      

    })

  })

  return can;

});

const areCardsHighlighted = computed(() => {

  let are = false;

  gridCellsPos.value.forEach((xArray, yIndex) => {

    xArray.forEach((cell, xIndex) => {

      if(!cell.card) return;

      if(cell.card.classList.includes('highlighted')) are = true;


    })

  })

  return are;

})

const selectedCard = ref<SelectedCardType>({card: null, refCard: null});
//const selectedCardCellRef = useTemplateRef('selectedCardCellRef');
//const selectedCardCellsPos = ref<GridCellInternal[][]>([[]]);

const resizeSelCard : SelectedCardType = {card: null, refCard: null};


const onSelectedCardInitialMousePosition = {y: -1, x: -1, cardY: -1, cardX: -1};
const onResizeCardInitialMousePosition = {y: -1, x: -1};




const cardsRef = ref<HTMLElement[][]>(initializeGridArrayRef());

const gridSizeX = 6;
const gridSizeY = 3;






function initializeGridArrayRef(){


  const tempArray = [[]]
  tempArray.length = gridSizeY;
  
  for(let y=0; y<gridSizeY; y++){
    tempArray[y] = [];
  }

  return tempArray;


}

function initializeEmptyGridArray(){

  let tempArray : GridCellInternal[][]= [[]];

  for(let i=0; i < gridSizeY; i++){

    tempArray[i] = [];

    for(let k=0; k < gridSizeX; k++){

      tempArray[i][k] = {style: []};

    }

  }

  return tempArray;

}

function initializeCardPos(){

  let tempArray : CardInternal[][]= [[]];

  for(let i=0; i < gridSizeY; i++){

    tempArray[i] = [];

  }

  return tempArray;

}

function createCards(){

  


  //gridCellsPos.value[0][0] = {text: "test1", classList: [], style: []};
  //gridCellsPos.value[1][0] = {text: "test2", classList: [], style: []};
  
  gridCellsPos.value[0][0].card = {text: "test1", classList: [], style: []};
  gridCellsPos.value[1][3].card = {text: "test2", classList: [], style: []};
  


}

createCards();



function setCardsInitialStyle(){

  

}





function emptyStyleHandler(yIndex : number, xIndex : number, gridCell : GridCellInternal){

  return {};

  /*
  if(!gridCell.card) return {};

  const left = gridCellsPos.value[yIndex][xIndex - 1] ? gridCellsPos.value[yIndex][xIndex - 1] : null;
  const top = gridCellsPos.value[yIndex - 1] ? gridCellsPos.value[yIndex - 1 ][xIndex] : null;

  if(
    left?.card && left.card == gridCell.card ||
    top?.card && top.card == gridCell.card) return {};


  let right : GridCellInternal;
  let bottom : GridCellInternal;

  let lowestCardIndex : number = 0;
  let rightestCardIndex : number = 0;

  for(let YInd=0; YInd<gridSizeY; YInd++){

    const fCell = gridCellsPos.value[YInd][xIndex];

    if(fCell.card && fCell.card == gridCell.card) {
      bottom = gridCellsPos.value[YInd][xIndex];
      lowestCardIndex = YInd;
    }

  }

  for(let XInd=0; XInd<gridSizeX; XInd++){

    const fCell = gridCellsPos.value[yIndex][XInd];
    
    if(fCell.card && fCell.card == gridCell.card) {
      right = gridCellsPos.value[yIndex][XInd];
      rightestCardIndex = XInd;
    }

  }

  const rowSpan = 1 + lowestCardIndex - yIndex;
  const columnSpan = 1 + rightestCardIndex - xIndex;

  //if(xIndex == 0 && yIndex == 1) console.log(rowSpan, columnSpan);

  return {'grid-row': 'span '+rowSpan, 'grid-column': 'span '+ columnSpan};
  */
}


function emptyVIFHandler(yIndex : number, xIndex : number, gridCell : GridCellInternal){

  if(!gridCellsPos.value[yIndex][xIndex].card) return true;


  const left = gridCellsPos.value[yIndex][xIndex - 1];
  const top = gridCellsPos.value[yIndex - 1] ? gridCellsPos.value[yIndex - 1][xIndex] : null;

  if(left?.card || top?.card) return false;


  return true;

}

function cardVIFHandler(yIndex: number, xIndex: number){


  if(!gridCellsPos.value[yIndex][xIndex].card) return false;


  const card = gridCellsPos.value[yIndex][xIndex].card;

  const leftCard = gridCellsPos.value[yIndex][xIndex-1] ? gridCellsPos.value[yIndex][xIndex-1].card : null;
  const TopCard = gridCellsPos.value[yIndex-1] ? gridCellsPos.value[yIndex-1][xIndex].card : null;

  if(
    leftCard && leftCard == card ||
    TopCard && TopCard == card
  ) return false;


  return true;



}

type DynamicObj = {
  [key: string]: any;
}

function cardStyleHandler(card?: CardInternal | null){

  const obj : DynamicObj = {};

  if(!card) return;
  

  card.style.forEach((style, index) => {
    if(!style) return;

    obj[style?.key] = style.value;

  });

  return obj;
}


function selectedCardCellStyleHandler(){

  const card = selectedCard.value.card;
  const cardRef = selectedCard.value.refCard;

  if(!cardRef || !card) return {};
  if(!gridRef.value) return {};


  const cardBoundRec = cardRef.getBoundingClientRect();
  const gridBoundRec = gridRef.value.getBoundingClientRect();

  const height = cardBoundRec.height;
  const width = cardBoundRec.width;

  const left = cardBoundRec.left;
  const top = cardBoundRec.top;

  const style = {
    position: 'fixed',
    height: height+'px',
    width: width+'px',
    left: left+'px',
    top: top+'px',
    'z-index': '10',
  };

  const keys = Object.keys(style);
  const values = Object.values(style);

  for(let i = 0; i<Object.keys(style).length; i++){
    
    if(card.style.find((el) => el.key == keys[i])) continue;

    card.style.push({key: keys[i], value: values[i]});
    
    
  }
  
  let styleToReturn : DynamicObj = {}; 

  card.style.forEach((style, index) => {

    styleToReturn[style.key] = style.value;

  });


  return styleToReturn;
  

}


function selectedCardClassHandler(){

  const card = selectedCard.value.card;

  if(!card) return [];

  return card.classList;




}


function cardMouseDownLeftHandler(yIndex : number, xIndex : number, mouseEvent : MouseEvent){

  const card = gridCellsPos.value[yIndex][xIndex].card;
  const cardRef = cardsRef.value[yIndex][xIndex] as HTMLElement;

  if(!card) return;

  if(card.classList.includes('resizable')){
    /*
    cardSetResizing(card, cardRef , mouseEvent)

    bodyMouseUpUseFunc((mEvent : MouseEvent) => cardUnsetResizing(yIndex, xIndex, card, cardRef, mEvent), cardUnsetResizing.toString());
    */
  }else{

    cardSelect(yIndex, xIndex, mouseEvent, card);

    bodyMouseUpUseFunc((mEvent : MouseEvent) => cardUnselect(yIndex, xIndex, card, cardRef), cardUnselect.toString());

  }



}

function cardMouseDownRightHandler(yIndex : number, xIndex : number, mouseEvent : MouseEvent){

  const card = gridCellsPos.value[yIndex][xIndex].card;

  if(!card) return;

  cardToggleClass(card, 'highlighted');




}

function cardToggleClass(card : CardInternal, className : string){

  if(card.classList.includes(className)) cardRemoveClass(card, className);
  else cardAddClass(card, className);

}


function cardAddClass(card : CardInternal, className : string){

  if(!card.classList.includes(className)) card.classList.push(className);


}

function cardRemoveClass(card : CardInternal, className: string){

  if(card.classList.includes(className)) card.classList = card.classList.filter((c) => c != className);

}

/*
function cardSetResizing(card : CardInternal, cardRef : HTMLElement, mouseEvent : MouseEvent){



  if(!card.classList.includes('resizing')) card.classList.push('resizing');
  

  onResizeCardInitialMousePosition.y = mouseEvent.clientY;
  onResizeCardInitialMousePosition.x = mouseEvent.clientX;

  resizeSelCard.card = card;
  resizeSelCard.refCard = cardRef as HTMLElement;


  bodyMouseMoveUseFunc((mEvent : MouseEvent) => cardResizeHandler(mEvent), cardResizeHandler.toString());


}
*/

function cardSelect(yIndex : number, xIndex : number, mouseEvent : MouseEvent, card: CardInternal){

  selectedCard.value.card = card;
  selectedCard.value.refCard = cardsRef.value[yIndex][xIndex] as HTMLElement;
  
  if(!card.classList.includes('selected')) card.classList.push('selected');



  const cardRefBoundRec = selectedCard.value.refCard.getBoundingClientRect();


  onSelectedCardInitialMousePosition.x = mouseEvent.clientX;
  onSelectedCardInitialMousePosition.y = mouseEvent.clientY;
  onSelectedCardInitialMousePosition.cardY = cardRefBoundRec.top;
  onSelectedCardInitialMousePosition.cardX = cardRefBoundRec.left;





  bodyMouseMoveUseFunc((mEvent : MouseEvent) => cardMouseDragHandler(yIndex,xIndex, mEvent), cardMouseDragHandler.toString());


}



function cardMouseUpLeftEvHandler(yIndex : number, xIndex : number, mouseEvent : MouseEvent){

  if(selectedCard.value.card == null) return;

  const card = selectedCard.value.card;
  const cardRef = selectedCard.value.refCard as HTMLElement;

  if(card.classList.includes('selected')){
    cardUnselect(yIndex, xIndex, card, cardRef);
    return;
  }

  /*
  if(card.classList.includes('resizing')){
    cardUnsetResizing(yIndex, xIndex, card, cardRef, mouseEvent);
  }
  */

}


function cardUnselect(yIndex : number, xIndex : number , card : CardInternal, cardRef : HTMLElement){

  card.classList = card.classList.filter((c) => c != 'selected');

  let closestEmptySpaceX = -1;
  let closestEmptySpaceY = -1;

  let closestCenterXDist : number;
  let closestCenterYDist : number;

  const cardRefBoundRec = cardRef.getBoundingClientRect();

  gridCellsRef.value.forEach((xArray, yInd) => {

    xArray.forEach((emptySpace, xInd) => {

      if(emptySpace.classList.contains('proximity')){
        
        const emptyBoundRec = emptySpace.getBoundingClientRect();

        const centerXDistance = Math.abs((cardRefBoundRec.left + cardRefBoundRec.width / 2) - (emptyBoundRec.left + emptyBoundRec.width / 2));
        const centerYDistance = Math.abs((cardRefBoundRec.top + cardRefBoundRec.height / 2) - (emptyBoundRec.top + emptyBoundRec.height / 2));


        if(!closestCenterXDist) closestCenterXDist = centerXDistance;
        if(!closestCenterYDist) closestCenterYDist = centerYDistance;


        if(closestCenterXDist >= centerXDistance){
          closestEmptySpaceX = xInd;
          closestCenterXDist = centerXDistance;
        }

        if(closestCenterYDist >= centerYDistance){
          closestEmptySpaceY = yInd;
          closestCenterYDist = centerYDistance;
        }
        
        

        emptySpace.classList.remove('proximity');

      }

      

    });

  });

  if(closestEmptySpaceX != -1 && closestEmptySpaceY != -1){
    //cardRearrange(yIndex, xIndex, closestEmptySpaceY, closestEmptySpaceX, card);

    let tempArray : GridCellInternal[][] = [[]];

    gridCellsPos.value.forEach((xArray, yIndex) => {

      tempArray[yIndex] = xArray.slice();

    });

    
    const elementA = gridCellsPos.value[yIndex][xIndex];
    const elementB = gridCellsPos.value[closestEmptySpaceY][closestEmptySpaceX];
    
    gridCellsPos.value[yIndex][xIndex] = elementB;
    gridCellsPos.value[closestEmptySpaceY][closestEmptySpaceX] = elementA;
    
    //console.log(yIndex,xIndex,' | ',closestEmptySpaceY,closestEmptySpaceX, elementA, elementB);
    

  }


  
  bodyMouseMoveRemoveFunc(cardMouseDragHandler.toString());
  bodyMouseUpRemoveFunc(cardUnselect.toString());
  
  cardRef.style.top = '0px';
  cardRef.style.left = '0px';
  
  selectedCard.value.card = null; 
  selectedCard.value.refCard = null; 
  

}

/*
function cardRearrange(yIndex : number, xIndex : number, closestEmptySpaceY : number, closestEmptySpaceX : number, card : CardInternal){

  
  if(!gridCellsPos.value[closestEmptySpaceY + 1]) closestEmptySpaceY--;
  if(!gridCellsPos.value[closestEmptySpaceY][closestEmptySpaceX + 1]) closestEmptySpaceX--;


  if(gridCellsPos.value[yIndex+1] && gridCellsPos.value[yIndex + 1][xIndex].card == card){

    if(!gridCellsPos.value[closestEmptySpaceY+1]) return;

    cardRearrange(yIndex+1, xIndex, closestEmptySpaceY+1, closestEmptySpaceX, card);

  }

  if(gridCellsPos.value[yIndex][xIndex+1] && gridCellsPos.value[yIndex][xIndex+1].card == card){

    if(!gridCellsPos.value[closestEmptySpaceY][closestEmptySpaceX+1]) return;

    cardRearrange(yIndex, xIndex+1, closestEmptySpaceY, closestEmptySpaceX+1, card);

  }



  let tempArray : GridCellInternal[][] = [[]];
  
  gridCellsPos.value.forEach((xArray, yInd) => {

    tempArray[yInd] = xArray.slice();

  });




  for(
    let Xaxis = xIndex < closestEmptySpaceX ? xIndex-1 : xIndex+1,
    Yaxis = yIndex < closestEmptySpaceY ? yIndex-1 : yIndex+1;
    Xaxis != closestEmptySpaceX || Yaxis != closestEmptySpaceY;){

    if(Xaxis < closestEmptySpaceX) Xaxis++;
    else if(Xaxis > closestEmptySpaceX) Xaxis--;
    if(Yaxis < closestEmptySpaceY) Yaxis++;
    else if(Yaxis > closestEmptySpaceY) Yaxis--;


    console.log(Xaxis,Yaxis);

    if(Xaxis != closestEmptySpaceX){


    }

    if(Yaxis != closestEmptySpaceY){



    }
    


    
  }
  console.log('--');
  


  gridCellsPos.value = tempArray;

  //const elementA = gridCellsPos.value[closestEmptySpaceY][closestEmptySpaceX];
  //const elementB = gridCellsPos.value[yIndex][xIndex];

  
  //gridCellsPos.value[closestEmptySpaceY][closestEmptySpaceX] = elementB
  //gridCellsPos.value[yIndex][xIndex] = elementA;

}
*/

/*
function cardUnsetResizing(yIndex : number, xIndex : number , card : CardInternal, cardRef : HTMLElement, mouseEvent : MouseEvent){

  card.classList = card.classList.filter((c) => c != 'resizing');


  const cardRefBoundRect = cardRef.getBoundingClientRect();


  const isXColliding = mouseEvent.clientX >= cardRefBoundRect.left && mouseEvent.clientX <= cardRefBoundRect.right;
  const isYColliding = mouseEvent.clientY >= cardRefBoundRect.top && mouseEvent.clientY <= cardRefBoundRect.bottom;

  
  if(!isXColliding && !isYColliding) card.classList = card.classList.filter((c) => c != 'resizable');



  gridCellsRef.value.forEach((xArray, yIndex) => {

    xArray.forEach((empty, xIndex) => {

      if(empty.classList.contains('proximityResize')){

        gridCellsPos.value[yIndex][xIndex].card = card;

      }

    });

  });


  resizeSelCard.card = null;
  resizeSelCard.refCard = null;


  bodyMouseMoveRemoveFunc(cardResizeHandler.toString());
  bodyMouseUpRemoveFunc(cardUnsetResizing.toString());


}
*/


function cardMouseMoveEvHandler(yIndex : number, xIndex : number, mouseEvent : MouseEvent){

  return;
  //future resizable cards

  /*
  if(selectedCard.value.card) return;
  

  

  const cardRef = cardsRef.value[yIndex][xIndex];
  const cardBoundRec = cardRef.getBoundingClientRect();
  const card = gridCellsPos.value[yIndex][xIndex].card;

  if(!card) return;


  const centerXDistance = Math.abs((cardBoundRec.left + cardBoundRec.width / 2) - (mouseEvent.clientX));
  const centerYDistance = Math.abs((cardBoundRec.top + cardBoundRec.height / 2) - (mouseEvent.clientY));

  const collisionXSize = (cardBoundRec.width / 2) - 20;
  const collisionYSize = (cardBoundRec.height / 2) - 20;

  const isXColliding = centerXDistance < collisionXSize;
  const isYColliding = centerYDistance < collisionYSize;


    
  if(!isXColliding || !isYColliding){

    if(!card.classList.includes('resizable')) card.classList.push('resizable');
    
    
  }else{
    
    card.classList = card.classList.filter((clas) => clas != 'resizable');

  }  

  */

}


/*
function cardResizeHandler(mouseEvent : MouseEvent){

  if(!resizeSelCard.refCard) return;

  const cardRef = resizeSelCard.refCard;

  const cardRefStyle = cardRef.style;

  const cardRefBoundRec = cardRef.getBoundingClientRect();




  const newWidth = Math.abs(cardRefBoundRec.left - mouseEvent.clientX);
  const newHeight = Math.abs(cardRefBoundRec.top - mouseEvent.clientY);



  cardRefStyle.width = newWidth.toString()+'px';
  cardRefStyle.height = newHeight.toString()+'px';


  calculateCardCollision(cardRef, 'proximityResize', -40);


}
*/


function cardMouseDragHandler(yIndex : number, xIndex : number, mouseEvent : MouseEvent){

  if(!selectedCard.value.card || !gridRef.value || !selectedCard.value.refCard) return;

  
  const card = selectedCard.value.card;
  const cardRef = selectedCard.value.refCard;
  const gridReff = gridRef.value;

  const gridRefBoundRec = gridReff.getBoundingClientRect();
  
  /*
  const newYPos = mouseEvent.clientY + (onSelectedCardInitialMousePosition.cardY - onSelectedCardInitialMousePosition.y);
  const newXPos = mouseEvent.clientX + (onSelectedCardInitialMousePosition.cardX - onSelectedCardInitialMousePosition.x);
  //const newXPos = mouseEvent.clientX;

  
  card.style = card.style.map((el) => {
    

    if(el.key == 'left') return {key: el.key, value:newXPos.toString()+'px'};
    if(el.key == 'top') return {key: el.key, value:newYPos.toString()+'px'};

    return el;
  });
  */

  const newYPos = mouseEvent.clientY - onSelectedCardInitialMousePosition.y;
  const newXPos = mouseEvent.clientX - onSelectedCardInitialMousePosition.x;


  cardRef.style.top = newYPos.toString()+'px';
  cardRef.style.left = newXPos.toString()+'px';
  

  const cardCollidingClass = 'proximity';
  const cardCollisionOffset = -40;

  calculateCardCollision(cardRef, cardCollidingClass, cardCollisionOffset);

}

function cardMouseLeaveEvHandler(yIndex : number, xIndex : number, mouseEvent : MouseEvent){
  
  const card = gridCellsPos.value[yIndex][xIndex].card;

  if(!card) return;

  if(card == selectedCard.value.card || card == resizeSelCard.card) return;


  card.classList = card.classList.filter((c) => {

    if(c == 'highlighted') return c;
    return;

  });

}




function calculateCardCollision(cardRef : HTMLElement, classToMark : string, collisionOffset = 0){


  gridCellsRef.value.forEach((xArray, yInd) => {

    xArray.forEach((emptySpace, xInd) => {
;
      
      const isColliding = boxCollision(cardRef, emptySpace, collisionOffset);

      if(isColliding){
        
        emptySpace.classList.add(classToMark);
        
      }else{
        
        emptySpace.classList.remove(classToMark);

      }

      



    })

  })



}


function boxCollision(objA : HTMLElement, objB : HTMLElement, objAOffset = 0, objBOffset = 0) : boolean {

  const ABoundRec = objA.getBoundingClientRect();
  const BBoundRec = objB.getBoundingClientRect();

  if(ABoundRec.right + objAOffset >= BBoundRec.left - objBOffset && ABoundRec.left - objAOffset <= BBoundRec.right + objBOffset){
    if(ABoundRec.bottom + objAOffset >= BBoundRec.top - objBOffset && ABoundRec.top - objAOffset <= BBoundRec.bottom + objBOffset){
      return true;
    }
  }

  return false;
}


type MouseFuncObjWrapper = {

  funcs: {
    func?: (mouseEvent : MouseEvent) => void;
    id?: string;
  }[]

}


const bodyMouseMoveFuncs : MouseFuncObjWrapper = {funcs: []};

function bodyMouseMoveUseFunc(funcf : (mouseEvent : MouseEvent) => void, idf: string){


  if(bodyMouseMoveFuncs.funcs.find((f) => f.id == idf)) return;

  bodyMouseMoveFuncs.funcs.push({func: funcf, id: idf});

}

function bodyMouseMoveRemoveFunc(idf: string){



  if(!bodyMouseMoveFuncs.funcs.find((f) => f.id == idf)) return;


  bodyMouseMoveFuncs.funcs = bodyMouseMoveFuncs.funcs.filter((f) => f.id != idf);

}


function bodyMouseMoveEvHandler(mouseEvent:MouseEvent){

  bodyMouseMoveFuncs.funcs.forEach((fun, index) => {

    if(fun.func) fun.func(mouseEvent);

  });

}



const bodyMouseUpFuncs : MouseFuncObjWrapper = {funcs: []};


function bodyMouseUpUseFunc(funcf : (mouseEvent : MouseEvent) => void, idf: string) {

  if(bodyMouseUpFuncs.funcs.find((f) => f.id == idf)) return;

  bodyMouseUpFuncs.funcs.push({func: funcf, id: idf});

}

function bodyMouseUpRemoveFunc(idf : string) {

  if(!bodyMouseUpFuncs.funcs.find((f) => f.id == idf)) return;

  bodyMouseUpFuncs.funcs = bodyMouseMoveFuncs.funcs.filter((f) => f.id != idf);

}

function bodyMouseUpEvHandler(mouseEvent : MouseEvent){

  bodyMouseUpFuncs.funcs.forEach((fun, index) => {

    if(fun.func) fun.func(mouseEvent);

  });

}


function addCardEvHandler(){
  addCard();


}

function addCard(card? : CardInternal){

  if(!card) card = {text:'card Created', classList: [], style: []};


  let cardCreated = false;

  for(let yIndex = 0; yIndex<gridCellsPos.value.length; yIndex ++){
    if(cardCreated) break;


    const xArrayLength = gridCellsPos.value[yIndex].length;

    for(let xIndex = 0; xIndex<xArrayLength; xIndex++){

      if(!gridCellsPos.value[yIndex][xIndex].card){

        gridCellsPos.value[yIndex][xIndex].card = card;
        cardCreated = true;
      } 

      if(cardCreated) break;

    }


  }
  
}

function deleteCardsEvHandler(){

  gridCellsPos.value.forEach((xArray, yIndex) => {

    xArray.forEach((cell, xIndex) => {

      if(!cell.card) return;

      if(cell.card.classList.includes('highlighted')) {

        if(cell.card.img) URL.revokeObjectURL(cell.card.img.src);

        cell.card = null;
      }



    })

  });


}


function addCustomCardEvHandler(){

  customCardInput.click();

}

function onIncomingCard(event : Event){

  const files = (event.target as HTMLInputElement).files;

  if(!files) return;

  const file = files[0];
  const url = URL.createObjectURL(file);


  const img = document.createElement('img');
  img.src = url;

  const newCard : CardInternal = {classList:[], style:[],text: img.src, img: img}; 

  addCard(newCard);



}



</script>

<template>

  <div id="main">

    <div class="gridMain" ref="grid">
  
      <template
      v-for="(emptyRowArray,yIndex) in gridCellsPos">
        
        <template
        v-for="(empty, xIndex) in emptyRowArray" :key="xIndex.toString()+','+yIndex.toString()"
        
        >
          
  
          <div class="empty"
    
          :style="emptyStyleHandler(yIndex, xIndex, empty)"
    
          :ref="(el) => {if(el) gridCellsRef[yIndex][xIndex] = el as HTMLElement}"
          >
    
            <div class="card"
    
            :class="gridCellsPos[yIndex][xIndex].card?.classList"
    
            v-if="cardVIFHandler(yIndex, xIndex)"
            :style="cardStyleHandler(gridCellsPos[yIndex][xIndex].card)"
            
            @mousedown.left.prevent="(e: MouseEvent) => cardMouseDownLeftHandler(yIndex,xIndex, e)"
            @mouseup.left.prevent="(e: MouseEvent) => cardMouseUpLeftEvHandler(yIndex,xIndex, e)"
            @click.right.prevent="(e: MouseEvent) => cardMouseDownRightHandler(yIndex,xIndex,e)"
            @mousemove.prevent="(e: MouseEvent) => cardMouseMoveEvHandler(yIndex, xIndex, e)"
            @mouseleave.prevent="(e: MouseEvent) => cardMouseLeaveEvHandler(yIndex,xIndex,e)"
            
            :ref="(el) => {if(el) cardsRef[yIndex][xIndex] = el as HTMLElement}"
            >
    
              <h2>{{ xIndex }}, {{ yIndex }}</h2>
              <h2 v-if="!gridCellsPos[yIndex][xIndex].card?.img">{{ gridCellsPos[yIndex][xIndex].card?.text }}</h2>
              <img v-else :src="gridCellsPos[yIndex][xIndex].card.img.src"></img>


            </div>
      
          </div>
        </template>
  
      </template>
  
  
  
  
    </div>

    <div id="buttonsContainer">

      <div id="addRemoveWrapper">
        <button class="cardAction" id="addCardBtn"
        @click="addCardEvHandler()"
        :disabled="!canAddCard"
        
        > Add Card </button>
        <button class="cardAction" id="deleteCardBtn"
        @click="deleteCardsEvHandler()"
        :disabled="!areCardsHighlighted"
  
        > Delete Selected Cards </button>

      </div>


      <button class="cardAction" id="uploadImgBtn"
      
      @click="addCustomCardEvHandler()"
      :disabled="!canAddCard"

      > Upload Custom Image </button>

    </div>

  </div>

  

</template>

<style>

  button{
    height: fit-content;
  }

  #main{

    height: 100%;
    width: 100%;

    display: grid;

    grid-template-columns: 1fr auto 1fr;

    

    align-items: center;
    justify-items: center;

    justify-content: center;

    gap: 2rem;


    border: 2px solid orange;

  }

  #main > *{
    border: 2px solid green;

  }

  #buttonsContainer {

    width: 100%;
    height: 100%;


    display: flex;

    flex-direction: column;

    align-items: center;
    justify-content: center;

    grid-template-rows: 1fr 1fr;

 

    gap: 10rem;

  }


  #addRemoveWrapper {

    display: flex;

    align-items: center;
    justify-content: center;

    gap: 2rem;


  }

  button.cardAction {

    height: 5rem;
    width: 20rem;

    font-size: medium;
    font-weight: 600;

  }

  #addCardBtn {
    background-color: green;

  }

  #deleteCardBtn {
    background-color: brown;

  }

  #uploadImgBtn {
    background-color: aquamarine;

    align-self: center;
    justify-self: center;


  }


  .gridMain{
    
    height: 40rem;
    width: 100rem;
    
    grid-column: 2;

    background-color: darkgray;


    display: grid;

    grid-template-columns: repeat(6, 1fr);
    grid-template-rows: repeat(3, 1fr);

    gap: 0.5rem;

    padding: 0.5rem;



  }



  .gridSelectedCard{

    display: grid;

  }

  button.cardAction{



  }

  .empty{
    background-color: rgb(42, 42, 42);

    display: grid;

    grid-template-columns: 1fr;
    grid-template-rows: 1fr;

  }

  .empty.proximity {
    border: 2px solid white;
  }

  .empty.proximityResize {

    border: 2px solid cyan;

  }

  .card{
    background-color: darkkhaki;

    display: grid;
    padding: 0.5rem;

    grid-template-rows: 1fr 5fr;


    align-items: center;
    align-content: center;

    justify-items: center;

    border: 2px solid black;


  }

  .card > img {

    max-width: 100%;
    max-height: 100%;

  }

  .card.highlighted {
    background-color: aliceblue;
  }

  .card.selected{
    background-color: aquamarine;
    z-index: 10;
  }

  .card.resizable{
    border: 5px solid orange;
  }

  .card.resizing{
    border: 5px solid rgb(255, 222, 160);
    z-index: 10;
  }





</style>
