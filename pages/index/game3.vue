<template>
	<view class="container">
		<view class="board">
			<view v-for="(row, rowIndex) in board" :key="rowIndex" class="row">
				<view v-for="(cell, colIndex) in row" :key="colIndex" class="cell"
					@click="placeStone(rowIndex, colIndex)">
					<view v-if="cell !== 0" class="stone" :class="{'black': cell === 1, 'white': cell === -1}"></view>
				</view>
			</view>
		</view>
		<button @click="undo">悔棋</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				board: Array.from({
					length: 15
				}, () => Array(15).fill(0)),
				currentPlayer: 1, // 1 for black, -1 for white  
				history: []
			};
		},
		methods: {
			placeStone(row, col) {
				if (this.board[row][col] !== 0) return;

				this.board[row][col] = this.currentPlayer;
				this.history.push({
					board: [...this.board],
					currentPlayer: this.currentPlayer
				});
				this.switchPlayer();

				if (this.checkWin(row, col)) {
					alert(`${this.currentPlayer === 1 ? '黑棋' : '白棋'} 赢了！`);
				}
			},
			switchPlayer() {
				this.currentPlayer *= -1;
			},
			checkWin(row, col) {
				const directions = [{
						dx: 1,
						dy: 0
					}, {
						dx: 0,
						dy: 1
					},
					{
						dx: 1,
						dy: 1
					}, {
						dx: 1,
						dy: -1
					}
				];
				const player = this.board[row][col];
				const threshold = 5;

				for (const {
						dx,
						dy
					}
					of directions) {
					let count = 1;
					for (let i = 1; i < threshold; i++) {
						const newRow = row + i * dx;
						const newCol = col + i * dy;
						if (
							newRow >= 0 && newRow < 15 &&
							newCol >= 0 && newCol < 15 &&
							this.board[newRow][newCol] === player
						) {
							count++;
						} else {
							break;
						}
					}
					for (let i = 1; i < threshold; i++) {
						const newRow = row - i * dx;
						const newCol = col - i * dy;
						if (
							newRow >= 0 && newRow < 15 &&
							newCol >= 0 && newCol < 15 &&
							this.board[newRow][newCol] === player
						) {
							count++;
						} else {
							break;
						}
					}
					if (count >= threshold) return true;
				}
				return false;
			},
			undo() {
				if (this.history.length === 0) return;
				const {
					board,
					currentPlayer
				} = this.history.pop();
				this.board = board;
				this.currentPlayer = currentPlayer;
			}
		}
	};
</script>

<style scoped>
	.container {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: 100vh;
	}

	.board {
		display: grid;
		grid-template-columns: repeat(15, 40px);
		grid-gap: 2px;
		margin-bottom: 20px;
	}

	.row {
		display: grid;
		grid-template-columns: repeat(15, 40px);
	}

	.cell {
		width: 40px;
		height: 40px;
		background-color: #f0d9b5;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.stone {
		width: 36px;
		height: 36px;
		border-radius: 50%;
	}

	.black {
		background-color: black;
	}

	.white {
		background-color: white;
		border: 2px solid black;
	}

	button {
		padding: 10px 20px;
		font-size: 16px;
	}
</style>