<template>
  <div>
    <Scatter :data="data" :options="options" width="400" height="400" />
  </div>
</template>
<script setup lang="ts">
import { Chart, registerables } from 'chart.js';
import type { ChartData, ChartOptions } from 'chart.js';
import { computed, ref } from 'vue';
import { Scatter } from 'vue-chartjs';

Chart.register(...registerables);
const url = new URL(window.location.href);
const player1 = url.searchParams.get('player1') ?? 'Player 1';
const player2 = url.searchParams.get('player2') ?? 'Player 2';


type CellIndex = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8;
const unFilledCells = ref<CellIndex[]>([0, 1, 2, 3, 4, 5, 6, 7, 8])
const player1Cells = ref<CellIndex[]>([])
const player2Cells = ref<CellIndex[]>([])
const player1Color = 'rgb(54, 162, 235)'
const player2Color = 'rgb(255, 99, 132)'

const winningPlayer = computed(() => {
  if (getWinningCombo(player1Cells.value)) {
    return 'player1'
  }
  if (getWinningCombo(player2Cells.value)) {
    return 'player2'
  }
  if (unFilledCells.value.length === 0) {
    return 'draw'
  }
  return undefined
})

const winningCombo = computed(() => {
  return winningPlayer.value === 'player1' ? getWinningCombo(player1Cells.value) : getWinningCombo(player2Cells.value)
})

const nextPlayer = computed(() => {
  return player1Cells.value.length === player2Cells.value.length ? 'player1' : 'player2'
})

function cellIndexToCoord(cellIndex: CellIndex): { x: number, y: number } {
  return {
    x: cellIndex % 3 + 0.5,
    y: Math.floor(cellIndex / 3) + 0.5,
  }
}

function getWinningCombo(cells: CellIndex[]) {
  const winningCombos: [CellIndex, CellIndex, CellIndex][] = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ]
  return winningCombos.find((combo) => {
    return combo.every((cell) => cells.includes(cell))
  })
}

const data = computed<ChartData<"scatter">>(() => ({
  datasets: [
    {
      label: 'unFilledCells',
      data: unFilledCells.value.map(cellIndexToCoord),
      pointRadius: 20,
      pointHoverRadius: 20,
      backgroundColor: 'transparent',
      borderColor: 'rgba(0, 0, 0, 0.1)',
      borderWidth: 2,
      hoverBorderWidth: 2,
      hoverBorderColor: winningPlayer.value === undefined ? nextPlayer.value === 'player1' ? player1Color : player2Color : (ctx) => ctx.chart.data.datasets[0].borderColor,
      pointStyle: nextPlayer.value === 'player1' ? 'circle' : nextPlayer.value === 'player2' ? 'crossRot' : ''
    },
    {
      label: player1,
      data: player1Cells.value.map(cellIndexToCoord),
      pointStyle: 'circle',
      backgroundColor: 'transparent',
      borderColor: winningPlayer.value === undefined ? player1Color : winningPlayer.value === 'player1' ? player1Cells.value.map(i => winningCombo.value?.includes(i) ? player1Color : 'rgba(0, 0, 0, 0.3)') : 'rgba(0, 0, 0, 0.3)',
      borderWidth: 2,
      hoverBorderWidth: 2,
      pointRadius: 20,
      pointHoverRadius: 20,
    },
    {
      label: player2,
      data: player2Cells.value.map(cellIndexToCoord),
      pointStyle: 'crossRot',
      backgroundColor: 'transparent',
      borderColor: winningPlayer.value === undefined ? player2Color :winningPlayer.value === 'player2' ? player2Cells.value.map(i => winningCombo.value?.includes(i) ? player2Color : 'rgba(0, 0, 0, 0.3)') : 'rgba(0, 0, 0, 0.3)',
      borderWidth: 2,
      hoverBorderWidth: 2,
      pointRadius: 20,
      pointHoverRadius: 20,
    },
  ],
}));

const options = computed<ChartOptions<"scatter">>(() => ({
  animation: false,
  onClick: (e, elements) => {
    const elemsOfunFilledCell = elements.filter((element) => element.datasetIndex === 0)
    if (winningPlayer.value !== undefined || elemsOfunFilledCell.length === 0) {
      return
    }
    const cellIndex = unFilledCells.value[elements[0].index]
    if (nextPlayer.value === 'player1') {
      player1Cells.value.push(cellIndex)
    } else {
      player2Cells.value.push(cellIndex)
    }
    unFilledCells.value = unFilledCells.value.filter((cell) => cell !== cellIndex)
  },
  plugins: {
    title: {
      display: true,
      text: 'Tic Tac Toe',
    },
    tooltip: {
      enabled: false,
    },
    legend: {
      display: false,
    },
  },
  scales: {
    x: {
      max: 3,
      min: 0,
      type: 'linear',
      position: 'bottom',
      ticks: {
        display: false,
        maxTicksLimit: 4,
      },
      title: {
        display: true,
        text: "O " + player1 + (winningPlayer.value === "player1" ? " Wins!!" : ""),
        color: winningPlayer.value === undefined ? nextPlayer.value === 'player1' ? player1Color : 'rgba(0, 0, 0, 0.3)' : winningPlayer.value === 'player1' ? player1Color : 'rgba(0, 0, 0, 0.3)',
      }
    },
    y: {
      max: 3,
      min: 0,
      type: 'linear',
      position: 'bottom',
      ticks: {
        maxTicksLimit: 4,
        display: false,
      },
      title: {
        display: true,
        text: "X " + player2 + (winningPlayer.value === "player2" ? " Wins!!" : ""),
        color: winningPlayer.value === undefined ? nextPlayer.value === 'player2' ? player2Color : 'rgba(0, 0, 0, 0.3)' : winningPlayer.value === 'player2' ? player2Color : 'rgba(0, 0, 0, 0.3)',
      }
    },
  },
}));

</script>