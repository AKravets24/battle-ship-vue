<template>

  <div class="container">

    <div class="header flexed centred">

      <div class="flexed columned base-font">

        <stat-item text="Player shots"     :value="playerShots"/>
        <stat-item text="Successful shots" :value="playerSuccessfulShots"/>
        <stat-item text="Missed shots"     :value="playerMissedShots"/>
        <stat-item text="Accuracy"         :value="playerAccuracy" units="%" />
        <stat-item text="Last shot"        :value="playerLastShot.grid?.toUpperCase() || 'None'" />
        <stat-item text="Unfired cells"    :value="unShootedCellsPlayer.length" />
      </div>

      <div class="flexed columned base-font centred">

        <span>Round: {{ roundNumber }}</span>
        <span>Score</span>
        <span>{{ playerWinNumbers }} : {{ computerWinNumbers }} </span>
        <span>Shots:  {{ totalShots }} </span>
        <my-button v-show="!gameInProgress" @click="restartGame" class="small-font">
          {{ roundNumber === 0 ? 'Start game!' : ' Next round!' }}
        </my-button>
        <span v-show="!gameInProgress && winFlag !== undefined"
        :class="[winFlag ? 'silent' : 'alert']"
        >
          {{ winFlag ? 'You win!' : 'You lose!' }}
        </span>

      </div>

      <div class="flexed columned base-font">

        <stat-item text="AI shots"         :value="computerShots"           reversed />
        <stat-item text="Successful shots" :value="computerSuccessfulShots" reversed />
        <stat-item text="Missed shots"     :value="computerMissedShots"     reversed />
        <stat-item text="Accuracy"         :value="computerAccuracy"     reversed units="%"/>
        <stat-item text="Last shot"        :value="computerLastShot.grid?.toUpperCase() ||'None'" reversed />
        <stat-item text="Unfired cells"    :value="unShootedCellsComputer.length" reversed />
      </div>

    </div>

    <div class="game-body flexed centred">
      <battle-field
              operator="player"
              :playerFire="playerFire"
              :fieldOwnerObj="playerField"
               fieldOwner="playerField"
              :lastShot="computerLastShot"
              :cheatMode="cheatMode"
              :cursorCheat="cursorCheat"
              :cellDictNumbers="cellDictNumbers"
      />
      <battle-field
              operator="computer"
              :playerFire="playerFire"
              :fieldOwnerObj="computerField"
               fieldOwner="computerField"
              :lastShot="playerLastShot"
              :cheatMode="cheatMode"
              :cursorCheat="cursorCheat"
              :cellDictNumbers="cellDictNumbers"
      />


    </div>

    <div class="toolbar flexed centred">

      <statistics-modal  :totalStat="statisticsObj" v-if="statisticsShown" @closeModal="statModalToggler"
       @cheatEvent="cheatEvent"
      />

<!--      <my-button @click="restartGame">Restart game</my-button>-->

      <my-button @click="statModalToggler" :disabled="gameInProgress">Show statistics</my-button>

      <my-button
              v-show="cheatModeShown"
              :lightToggle="cheatMode"
              @click="cheatMode = !cheatMode"
      >Cheat mode: {{ cheatMode ? 'Enabled' : 'Disabled' }}
      </my-button>
      <my-button
              v-show="cursorCheatShown"
              :lightToggle="cursorCheat"
              @click="cursorCheat = !cursorCheat"
      >Cursor cheat: {{ cursorCheat ? 'Enabled' : 'Disabled' }}
      </my-button>

    </div>

    <div class="footer flexed centred">

    <div>
      <progress-bar :ships="playerShips" @gameStopped="gameStopped" shipsOwner="player"/>
      <div class="flexed justifyed">

        <ship-statuses v-for="(shipArr, name) in playerShips"
                       :shipArr="shipArr"
                       :damageCalc="damageCalc"
                       :shipType="name"
                       shipsOwner="playerShips"
        />
      </div>

    </div>

    <div>
      <progress-bar :ships="computerShips" @gameStopped="gameStopped" shipsOwner="computer" accuracy/>
      <div class="flexed justifyed">

        <ship-statuses v-for="(shipArr, name) in computerShips"
                       :shipArr="shipArr"
                       :damageCalc="damageCalc"
                       :shipType="name"
                       shipsOwner="computerShips"
        />
      </div>

    </div>

  </div>

  </div>


</template>

<script lang="ts">

  import {defineComponent} from 'vue';
  import ShipStatuses from '@/components/ShipStatuses.vue';
  import ProgressBar from '@/components/ProgressBar.vue';
  import MyButton from '@/components/Button.vue';
  import StatItem from "@/components/StatItem.vue";
  import BattleField from "@/components/BattleField.vue";
  import StatisticsModal from "@/components/StatisticsModal.vue";

  export interface fieldCell {
    grid: string, idx: number,
    isShip: boolean,
    shipState?: ShipState_Type,
    isAvaliable: boolean,
    shotted: boolean,
    adjacentShown?: boolean
  }

  export type Shooters_Type = 'player' | 'computer';

  export type Ships_Types = 'skiffs'| 'brigs' | 'warships' | 'manowars';
  type GameFields_Type = 'playerField' | 'computerField';
  export type ShipState_Type = { type: Ships_Types, alive: boolean, idx?: number, shipId: number }
  export type ShipOwner_Type = 'playerShips' | 'computerShips'
  export type ShipsDescr_Type = {
    skiffs:   ShipState_Type[][]
    brigs:    ShipState_Type[][]
    warships: ShipState_Type[][]
    manowars: ShipState_Type[][]
  }
  type AdjacentCells_Type = {
    [key: number] : {cellIdx: number, shown: boolean, shipType: Ships_Types}[]
  }
  type ServeInfoShot_Type = {
    idx?: number,
    state?:  'destroyed' | 'damaged' | undefined,
    direction?: 'column' | 'row' | undefined,
    shipArr?: number[],
  }

  type PersonStat_Type = {
    totalShots?:      number,
    successfulShots: number,
    missedShots:     number,
    accuracy?:        number,
    winNumbers?:      number,
    roundNum?:        number,
  }

  type DetailsType = {
    roundNumber: number, player: PersonStat_Type, computer: PersonStat_Type,
  }

  type Statistics_Type = {
    totalRounds: number,
    totalScore:  number,
    totalShots:  number,

    player:      PersonStat_Type,
    computer:    PersonStat_Type,

    rounds: DetailsType []
  }




export default defineComponent({

  components: {
    ShipStatuses, ProgressBar, MyButton, StatItem, BattleField, StatisticsModal,
  },

  data() {
    return {
      playerField: [] as fieldCell[],
      computerField: [] as fieldCell[],

      playerWinNumbers: 0 as number,
      computerWinNumbers: 0 as number,

      gameInProgress: false as boolean,
      winFlag: undefined as boolean | undefined,

      totalShots: 0 as number,

      cheatMode: false as boolean,
      cheatModeShown: false as boolean,
      cursorCheat: false as boolean,
      cursorCheatShown: false as boolean,

      statisticsObj: {
        totalRounds: 0 as number,
        totalShots:  0 as number,
        player: {
          accuracy:         0 as number,
          missedShots:      0 as number,
          successfulShots:  0 as number,
          totalShots:       0 as number,
          winNumbers:       0 as number,
        },
        computer: {
          accuracy:           0 as number,
          missedShots:        0 as number,
          successfulShots:    0 as number,
          totalShots:         0 as number,
          winNumbers:         0 as number,
        },
        rounds: [] as DetailsType[],
      } as Statistics_Type,
      statisticsShown: false as boolean,

      roundNumber: 0 as number,

      playerShots: 0 as number,
      playerSuccessfulShots: 0 as number,
      playerMissedShots: 0 as number,
      playerLastShot: {} as fieldCell,

      computerShots: 0 as number,
      computerSuccessfulShots: 0 as number,
      computerMissedShots: 0 as number,
      computerLastShot: {} as fieldCell,
      computerLastSuccessShot: { shipArr: [] } as unknown as ServeInfoShot_Type,

      moveBy: 'player' as Shooters_Type,

      cellDictLetters: {'a': 1, 'b': 2, 'c': 3, 'd': 4, 'e': 5, 'f': 6, 'g': 7, 'h': 8, 'i': 9, 'j': 10,},
      cellDictNumbers: {1: 'a', 2: 'b', 3: 'c', 4: 'd', 5: 'e', 6: 'f', 7: 'g', 8: 'h', 9: 'i', 10: 'j',}, // юзается

      playerShips: {
        skiffs: [],
        brigs: [],
        warships: [],
        manowars: [],
      } as ShipsDescr_Type,
      myAdjacentCells: {} as AdjacentCells_Type,

      computerShips: {
        skiffs: [],
        brigs: [],
        warships: [],
        manowars: [],
      } as ShipsDescr_Type,
      enemyAdjacentCells: {} as AdjacentCells_Type,
    }
  },

  methods: {

    statModalToggler () {
      return this.statisticsShown = !this.statisticsShown
    },

    cheatEvent (event: any) {
      if(event.cursorCheat) { this.cursorCheatShown = true }
      if(event.cheatMode)   { this.cheatModeShown   = true }
    },

    gameStopped (flag: boolean) {
      this.gameInProgress = false;
      flag ? this.playerWinNumbers ++ : this.computerWinNumbers++;
      this.winFlag = flag;
      this.cheatMode = true;

      let statCopy = {...this.statisticsObj};

      statCopy.totalShots += this.totalShots;
      statCopy.totalRounds = this.roundNumber;

      flag ? statCopy.player.winNumbers! ++ : statCopy.computer.winNumbers! ++;

      statCopy.player.totalShots!       += this.playerShots;
      statCopy.player.successfulShots   += this.playerSuccessfulShots;
      statCopy.player.missedShots       += this.playerMissedShots;

      statCopy.computer.totalShots!     += this.computerShots;
      statCopy.computer.successfulShots += this.computerSuccessfulShots;
      statCopy.computer.missedShots     += this.computerMissedShots;

      let roundStat = {
        roundNumber: this.roundNumber,
        player: {
          totalShots:       this.playerShots,
          successfulShots:  this.playerSuccessfulShots,
          missedShots:      this.playerMissedShots,
        },
        computer: {
          totalShots:       this.computerShots,
          successfulShots:  this.computerSuccessfulShots,
          missedShots:      this.computerMissedShots,
        }
      };

      statCopy.rounds.unshift(roundStat)

      this.statisticsObj = statCopy;

    },

    setInitGameState() {
      this.playerField = [];
      this.computerField = [];
      this.totalShots = 0;
      this.playerShots = 0;
      this.playerSuccessfulShots = 0;
      this.playerMissedShots = 0;
      this.playerLastShot = {}  as fieldCell;
      this.computerShots = 0;
      this.computerSuccessfulShots = 0;
      this.computerLastSuccessShot = { shipArr: [] };
      this.computerMissedShots = 0;
      this.computerLastShot = {}  as fieldCell;
      this.moveBy = 'player';

      this.playerShips = {
          skiffs: [],
          brigs: [],
          warships: [],
          manowars: [],
        };
      this.myAdjacentCells = {};
      this.computerShips = {
          skiffs: [],
          brigs: [],
          warships: [],
          manowars: [],
        };
      this.enemyAdjacentCells = {};
      this.cursorCheat = false;
      this.cheatMode = false;
      this.gameInProgress = true;


      this.createCellMask();
      this.setBattleField();

    },

    restartGame() {
      this.roundNumber ++
      this.setInitGameState()
    },

    damageCalc (shipsOwner: ShipOwner_Type, shipsType: Ships_Types, i: number) {
            return this[shipsOwner][shipsType][i].filter((el) => !el.alive).length;
    },

    openFire ( cell: fieldCell, fireBy: Shooters_Type ) {
      this.totalShots ++;

      const fireFlag = fireBy === 'player';

      const shipsOwner        = fireFlag  ? 'playerShips'           : 'computerShips',
          adjacentCellsOwner  = fireFlag  ? 'myAdjacentCells'       : 'enemyAdjacentCells',
          fieldOwner          = fireFlag  ? 'playerField'           : 'computerField',
          missedShotsOwner    = !fireFlag ? 'playerMissedShots'     : 'computerMissedShots',
          successShotsOwner   = !fireFlag ? 'playerSuccessfulShots' : 'computerSuccessfulShots',
          shotsOwner          = !fireFlag ? 'playerShots'           : 'computerShots',
          lastShotBy          = !fireFlag ? 'playerLastShot'        : 'computerLastShot';

      const fieldCopy      = JSON.parse(JSON.stringify([...this[fieldOwner] ]));
      const cellCopy       = JSON.parse(JSON.stringify( {...cell, shotted: true}));
      let successShotCopy  = JSON.parse(JSON.stringify( {...this.computerLastSuccessShot}));

      this[ lastShotBy ] = cell;

      this[ shotsOwner ] ++;

      if(cell.shipState) {
        this[successShotsOwner] ++;

        fieldCopy[cellCopy.idx] = {... fieldCopy[cellCopy.idx], shotted: true,  shipState: {... fieldCopy[cellCopy.idx].shipState, alive: false }  };

        const { type, shipId } : { type: Ships_Types, shipId: number } = cellCopy.shipState;

        const dataShipIdx = this[shipsOwner][type].findIndex(el => el[0].shipId === shipId);

        const dataShipEl  = this[shipsOwner][type][dataShipIdx].findIndex(({idx}) => idx === cellCopy.idx );

        this[shipsOwner][type][dataShipIdx][dataShipEl].alive = false;

        const isDestroyed = fieldCopy.filter(({shipState}: fieldCell) => shipState?.shipId === shipId).every(({shipState}: fieldCell) => !shipState!.alive );

        if(isDestroyed) {

          if(fireFlag) { this.computerLastSuccessShot = {...successShotCopy, direction: undefined, idx: cellCopy.idx, state: 'destroyed', shipArr: []} };

          this[adjacentCellsOwner][shipId].forEach(({cellIdx}) => fieldCopy[cellIdx] = {...fieldCopy[cellIdx], adjacentShown: true} );
          this[fieldOwner] = fieldCopy;
          return

        } else {

          if(fireFlag) {
            successShotCopy.idx = cellCopy.idx;
            successShotCopy.state = 'damaged';
            successShotCopy.shipArr.push(cellCopy.idx)
            this.computerLastSuccessShot = successShotCopy
          }

          cellCopy.shipState!.alive = false;
          this[fieldOwner][cellCopy.idx] = cellCopy
          return
        }
      }
      else {
        this[missedShotsOwner] ++;
        this[fieldOwner][cellCopy.idx] = cellCopy
        return
      }

    },

    playerFire(cell: fieldCell, fireBy: Shooters_Type) {
      if( cell.shotted || cell.adjacentShown || !this.gameInProgress  ) { return }
      this.openFire(cell, fireBy);
      this.moveBy = 'computer';
    },

    generateCelltoFire() {

      let cellIdx: number, selectedCell;
      let successShotCopy = {...this.computerLastSuccessShot}

      let fireCorrector = (shotIdx: number, firstShotIdx: number = -1): fieldCell => {
        const decade = (arg: number) => arg.toString().length === 1 ? '0' : arg.toString().substring(0, 1); //  '0'|| '1' || '2' ..., || '9'

        const crossCells = [
          successShotCopy.direction === 'row'    ? undefined : this.playerField[shotIdx - 10],
          successShotCopy.direction === 'column' ? undefined : decade(shotIdx) === decade(shotIdx - 1) ? this.playerField[shotIdx - 1] : undefined,
          successShotCopy.direction === 'column' ? undefined : decade(shotIdx) === decade(shotIdx + 1) ? this.playerField[shotIdx + 1] : undefined,
          successShotCopy.direction === 'row'    ? undefined : this.playerField[shotIdx + 10],
        ].filter((el) => el && !el.shotted && !el.adjacentShown);

        if(!crossCells.length && firstShotIdx !== -1) {
          return selectedCell = fireCorrector(firstShotIdx, -1)
        }

        const rand = Math.floor(Math.random() * crossCells.length);
        cellIdx = crossCells[rand]!.idx;

        cellIdx = this.unShootedCellsPlayer.findIndex((el) => el.idx === cellIdx);

        return this.unShootedCellsPlayer[cellIdx];
      };


      if(successShotCopy.state === 'damaged' && successShotCopy.shipArr!.length >= 2 ) { // определяем ориентировку корабля
        const [i1, i2] = successShotCopy.shipArr!;
        successShotCopy.direction =  (i1+1 === i2) || (i1-1 === i2)  ?  'row' : 'column';
        selectedCell = fireCorrector(successShotCopy.idx!, i1)
      }

      else if (successShotCopy.state === 'damaged' && !this.computerLastShot.isShip) { // попал, затем промазал
        selectedCell = fireCorrector(successShotCopy.idx!, -1)
      }

      else if (successShotCopy.state === 'damaged') { // первичное попадание
        selectedCell = fireCorrector(this.computerLastShot.idx, -1)
      }
      else {
        cellIdx = Math.floor(Math.random() * this.unShootedCellsPlayer.length );
        selectedCell = this.unShootedCellsPlayer[cellIdx];
      }

      this.computerLastSuccessShot = successShotCopy;
      this.openFire(selectedCell!, 'player');
      this.moveBy = 'player';
    },

    shipGenerator(size: number, shipType: Ships_Types, gameField: GameFields_Type): any {

      const orientation = Math.random() < 0.5; // горизонтально или вертикально
      const startPoint  = Math.floor(Math.random() * 100);

      const fieldFlag = gameField === 'playerField';

      const gameFieldCopy = [...this[gameField]];

      let secondPoint: number = startPoint + (orientation ? 1 : 10);
      let thirdPoint:  number = startPoint + (orientation ? 2 : 20);
      let fourthPoint: number = startPoint + (orientation ? 3 : 30);

      switch (size) {
        case 1:  { secondPoint = 0; thirdPoint = 0; fourthPoint = 0;  break; }
        case 2:  { thirdPoint  = 0; fourthPoint = 0;  break;}
        case 3:  { fourthPoint = 0; break; }
        default: { break; }
      }

      if(
          !gameFieldCopy[startPoint].isAvaliable
          || ( secondPoint && (!gameFieldCopy[secondPoint]?.isAvaliable || secondPoint % 10 === 0 || secondPoint > 99 ) )
          || ( thirdPoint  && (!gameFieldCopy[thirdPoint]?.isAvaliable  || thirdPoint  % 10 === 0 || thirdPoint  > 99 ) )
          || ( fourthPoint && (!gameFieldCopy[fourthPoint]?.isAvaliable || fourthPoint % 10 === 0 || fourthPoint > 99 ) )
      )
        { return this.shipGenerator(size, shipType, gameField) }

      const shipId = Math.random();

      const shipState = { isShip: true, isAvaliable: false, shipState: { type: shipType, alive: true, shipId } };

      const shipsArrCopy = [...this[fieldFlag ? 'playerShips' : 'computerShips'][shipType] ];

      const shipStatArr: ShipState_Type[] = [];

      this[gameField][startPoint]                     = {...gameFieldCopy[startPoint],  ...shipState}; shipStatArr.push({shipId, type: shipType, alive: true, idx: startPoint })
      if (secondPoint) { this[gameField][secondPoint] = {...gameFieldCopy[secondPoint], ...shipState}; shipStatArr.push({shipId, type: shipType, alive: true, idx: secondPoint })}
      if (thirdPoint)  { this[gameField][thirdPoint]  = {...gameFieldCopy[thirdPoint],  ...shipState}; shipStatArr.push({shipId, type: shipType, alive: true, idx: thirdPoint })}
      if (fourthPoint) { this[gameField][fourthPoint] = {...gameFieldCopy[fourthPoint], ...shipState}; shipStatArr.push({shipId, type: shipType, alive: true, idx: fourthPoint })}


      shipsArrCopy.push(shipStatArr);

      this[fieldFlag ? 'playerShips' : 'computerShips'][shipType] = shipsArrCopy;


      const leftCell1  = (startPoint -11),
            leftCell2  = (startPoint - 1),
            leftCell3  = (startPoint + 9),
            leftCell4  = orientation ? null : secondPoint ? secondPoint + 9 : null,
            leftCell5  = orientation ? null : thirdPoint  ? thirdPoint  + 9 : null,
            leftCell6  = orientation ? null : fourthPoint ? fourthPoint + 9 : null;

      const somePoint = fourthPoint || thirdPoint || secondPoint || startPoint || 0;

      const rightCell1 =  orientation ?  somePoint  - 9 : ( startPoint -  9 ),
            rightCell2 =  orientation ?  somePoint  + 1 : ( startPoint +  1 ),
            rightCell3 =  orientation ?  somePoint + 11 : ( startPoint + 11 ),
            rightCell4 =  orientation ? null : ( secondPoint ? secondPoint + 11 : null ),
            rightCell5 =  orientation ? null : ( thirdPoint  ? thirdPoint  + 11 : null ),
            rightCell6 =  orientation ? null : ( fourthPoint ? fourthPoint + 11 : null );

      const topCell1   = startPoint - 10,
            topCell2   = orientation ? ( secondPoint ? secondPoint - 10 : null) : null,
            topCell3   = orientation ? ( thirdPoint  ? thirdPoint  - 10 : null) : null,
            topCell4   = orientation ? ( fourthPoint ? fourthPoint - 10 : null) : null;


      const bottomCell1 = orientation ? startPoint + 10 :                    (!secondPoint && !thirdPoint && !fourthPoint) ?  startPoint  + 10 : null,
            bottomCell2 = (orientation && secondPoint) ? secondPoint + 10 :  (secondPoint && !thirdPoint && !fourthPoint)  ?  secondPoint + 10 : null,
            bottomCell3 = (orientation && thirdPoint ) ? thirdPoint  + 10 :  (thirdPoint && !fourthPoint)                  ?  thirdPoint  + 10 : null,
            bottomCell4 = (orientation && fourthPoint) ? fourthPoint + 10 :  (fourthPoint ? fourthPoint + 10 : null);

      const adjacentCopy =  {...this[fieldFlag ? 'myAdjacentCells' : 'enemyAdjacentCells'], [shipId]: []};


      if((leftCell1+1)%10 && leftCell1 >= 0)               { gameFieldCopy[leftCell1].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: leftCell1, shown: false, shipType}]}
      if((leftCell2+1)%10 && leftCell2 > -1)               { gameFieldCopy[leftCell2].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: leftCell2, shown: false, shipType}]}
      if((leftCell3+1)%10 && leftCell3 < 100)              { gameFieldCopy[leftCell3].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: leftCell3, shown: false, shipType}]}
      if(leftCell4 && (leftCell4+1)%10 && leftCell4 < 100) { gameFieldCopy[leftCell4].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: leftCell4, shown: false, shipType}]}
      if(leftCell5 && (leftCell5+1)%10 && leftCell5 < 100) { gameFieldCopy[leftCell5].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: leftCell5, shown: false, shipType}]}
      if(leftCell6 && (leftCell6+1)%10 && leftCell6 < 100) { gameFieldCopy[leftCell6].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: leftCell6, shown: false, shipType}]}


      if(topCell1 > -1)                { gameFieldCopy[topCell1].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: topCell1, shown: false, shipType}]}
      if(topCell2 && (topCell2 > -1))  { gameFieldCopy[topCell2].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: topCell2, shown: false, shipType}]}
      if(topCell3 && (topCell3 > -1))  { gameFieldCopy[topCell3].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: topCell3, shown: false, shipType}]}
      if(topCell4 && (topCell4 > -1))  { gameFieldCopy[topCell4].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: topCell4, shown: false, shipType}]}


      if(bottomCell1 && (bottomCell1 < 100))             { gameFieldCopy[bottomCell1].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: bottomCell1, shown: false, shipType}]}
      if(bottomCell2 && (bottomCell2 < 100)) { gameFieldCopy[bottomCell2].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: bottomCell2, shown: false, shipType}]}
      if(bottomCell3 && (bottomCell3 < 100)) { gameFieldCopy[bottomCell3].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: bottomCell3, shown: false, shipType}]}
      if(bottomCell4 && (bottomCell4 < 100)) { gameFieldCopy[bottomCell4].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: bottomCell4, shown: false, shipType}]}


      if(rightCell1 % 10 && rightCell1 > 0 )                   { gameFieldCopy[rightCell1].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: rightCell1, shown: false, shipType: shipType}]}
      if(rightCell2 % 10)                                      { gameFieldCopy[rightCell2].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: rightCell2, shown: false, shipType: shipType}]}
      if(rightCell3 % 10 && rightCell3 <= 100)                 { gameFieldCopy[rightCell3].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: rightCell3, shown: false, shipType: shipType}]}
      if(rightCell4 && (rightCell4 % 10 && rightCell4 <= 100)) { gameFieldCopy[rightCell4].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: rightCell4, shown: false, shipType: shipType}]}
      if(rightCell5 && (rightCell5 % 10 && rightCell5 <= 100)) { gameFieldCopy[rightCell5].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: rightCell5, shown: false, shipType: shipType}]}
      if(rightCell6 && (rightCell6 % 10 && rightCell6 <= 100)) { gameFieldCopy[rightCell6].isAvaliable = false;
        adjacentCopy[shipId] = [...adjacentCopy[shipId], {cellIdx: rightCell6, shown: false, shipType: shipType}]}

      this[fieldFlag ? 'myAdjacentCells' : 'enemyAdjacentCells'] = adjacentCopy;

    },

    setBattleField () {
      for(let x = 1; x <= 1; x++) {
        this.shipGenerator(4, 'manowars', 'playerField');
        this.shipGenerator(4, 'manowars', 'computerField');
      }

      for(let x = 1; x <= 2; x++) {
        this.shipGenerator(3, 'warships', 'playerField');
        this.shipGenerator(3, 'warships', 'computerField');
      }

      for(let x = 1; x <= 3; x++) {
        this.shipGenerator(2, 'brigs', 'playerField');
        this.shipGenerator(2, 'brigs', 'computerField');
      }

      for(let x = 1; x <= 4; x++) {
        this.shipGenerator(1, 'skiffs', 'playerField');
        this.shipGenerator(1, 'skiffs', 'computerField');
      }

    },

    createCellMask () {
      let counter = 0;
      for (let i = 1; i <= 10; i++) {
        for (let j = 1; j <= 10; j++) {
          this.playerField.push({grid: `${(this.cellDictNumbers as any)[i]} ${j}`,  isShip: false, idx: counter, isAvaliable: true, shotted: false})
          this.computerField.push({grid: `${(this.cellDictNumbers as any)[i]} ${j}`,isShip: false, idx: counter, isAvaliable: true, shotted: false})
          ++counter
        }
      }
    },

    unShootedSells(gameField: GameFields_Type) {
      return this[gameField].filter((el) => !el.adjacentShown && !el.shotted )
    }

  },

  computed: {
    unShootedCellsPlayer:   function(){ return this.unShootedSells('playerField')},
    unShootedCellsComputer: function(){ return this.unShootedSells('computerField')},

    playerAccuracy:         function () { return Math.round(this.playerSuccessfulShots   / this.playerShots   * 100) || 0 },
    computerAccuracy:       function () { return Math.round(this.computerSuccessfulShots / this.computerShots * 100) || 0 },

  },

  watch: {
    moveBy: function () {
      if(this.moveBy === 'computer') {
        this.generateCelltoFire()
      }
    }
  },

  created() {
    this.createCellMask();
  },

  mounted() {
    this.setBattleField();
  },

  props: {
    msg: String
  }

})
</script>

<style scoped lang="less">
  @baseColor: teal;
  @baseFontSize: 20px;
  @smallFontSize: 15px;
  @largeFontSize: 25px;


  .container {
    outline: 2px solid @baseColor;
  }

  .header {
    min-height: 100px;
  }

  .base-font {
    font-size: @baseFontSize;
  }
  .small-font {
    font-size:  @smallFontSize;
  }

  .game-body {
    /*height: 600px;*/
    padding: 10px 0;
    outline: 2px solid @baseColor;
  }

  .flexed {
    display: flex;
  }

  .justifyed {
    justify-content: space-evenly;
    flex-wrap: wrap;
  }

  .centred {
    justify-content: space-evenly;
    align-items: center;
  }

  .columned {
    flex-direction: column;
  }


  .footer > div {
    width: 50%;
  }

  .alert {
    background: firebrick;
  }

  .silent {
    background: lightgreen;
  }


</style>
