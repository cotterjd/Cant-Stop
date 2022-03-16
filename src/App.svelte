<h1>Can't Stop! &nbsp; &nbsp; <span style="color: {state.player === 1 ? 'blue' : 'red'};">Player {state.player} Turn</span></h1>
<!--<div class="debug">
  {JSON.stringify(state)}
</div>-->

{#each state.activeChips as chip}
	<span class="active-chip" 
				style="top: {chip.location[1]}px; left: {chip.location[0]}px"/>
  <span class="active-chip" 
				style="top: {chip.location[1]}px; left: {chip.location[0]}px"/>
  <span class="active-chip" 
				style="top: {chip.location[1]}px; left: {chip.location[0]}px"/>
{/each}

{#each Object.keys(state.player1Chips) as key}
  <span class="player1-chip" 
				style={getLocationStyles(key, `player1Chips`)} />
{/each}
{#each Object.keys(state.player2Chips) as key}
  <span class="player2-chip" 
				style={getLocationStyles(key, `player2Chips`)} />
{/each}


<div class="btn-container">
	<button on:click={lockInGains}>stop</button>
	<button on:click={roleDice} disabled={state.turnPhase==='choose'}>roll!</button>
	<div>
		Role:
		<span>{die1}</span>
		<span>{die2}</span>
		
		<span>{die3}</span>
		<span>{die4}</span>
		<div>
			<p>Possiblities</p>
			{#each ps as p }
				<div>
					<button on:click={moveChips(p)}>{p}</button>
				</div>
			{/each}
		</div>
	</div>
</div>

<div class="border" />

{#each ascCols as col }
  <div class="col-c" 
			 style="
							top: {ascTop-40*(col[0]-1)}px;
							left: {ascLeft+40*(col[0]-1)}px;
						"
			 >
		<div class="col" style="{state.colColors[col[0]]};">
			{#each col as x}
			  <span style="padding: 1px;">{x}</span>
			  &nbsp;
			{/each}
		</div>		
</div>

{/each}
{#each descCols as col }
  <div class="col-c" 
			 style="
							top: {descTop+40*(col[0]-1)}px;
							left: {descLeft+40*(col[0]-1)}px;
						"
	>
		<div class="col" style="{state.colColors[col[0]]};">
			{#each col as x}
			  <span style="padding: 1px;">{x}</span>
			  &nbsp;
			{/each}
		</div>		
</div>
{/each}

<script>
	let ps = []
	let die1 = `3`
	let die2 = `1`
	let die3 = `6`
	let die4 = `3`
  const chip1Home = [120, 100]
	, chip2Home = [87, 140]
	, chip3Home = [87, 100]
	, defaultChips = Object.freeze([Object.freeze({
		  id: 1,
		  isAvailable: true,
		  rope: 0,
		  position: 0,
		  location: chip1Home,
	  }),Object.freeze({
		  id: 2,
		  isAvailable: true,
		  rope: 0,
			position: 0,
		  location: chip2Home,
	  }),Object.freeze({
		  id: 3,
		  isAvailable: true,
		  rope: 0,
			position: 0,
		  location: chip3Home,
	  })])
	, defaultPlayerChips = Object.freeze({
		  1: 0,
		  2: 0,
		  3: 0,
		  4: 0,
		  5: 0,
		  6: 0,
		  7: 0,
		  8: 0,
		  9: 0,
		  10: 0,
		  11: 0,
		  12: 0,
	  })
	
	, getLocationStyles = (key, pStr) => {
		  const pChips = state[pStr]
			const loc = getLocation(key, pChips)
			if (pStr === `player1Chips`) {
				return `position:absolute; top: ${loc[1]}px; left: ${loc[0]-5}px;` 
			} else if (pStr === `player2Chips`) {
				return `position:absolute; top: ${loc[1]}px; left: ${loc[0]+5}px;` 
			}

	  }
	
	, getLocation = (key, playerChips) => {
		  if (playerChips[key] === 0) return [0, 0]
		  return ropePositions[key][playerChips[key]-1]
	  }
	
	, state = {
		  player: 1,
		  activeChips: [ ...defaultChips ],
		  player1Chips: { ...defaultPlayerChips },
		  player1Ropes: [], // deprecated
		  player2Chips: { ...defaultPlayerChips },
		  player2Ropes: [], //deprecated
			turnPhase: `roll`, // roll | choose
		  ownedRopes: [],
		  colColors: {
		    2: `border: 1px solid black`,
		    3: `border: 1px solid black`,
		    4: `border: 1px solid black`,
		    5: `border: 1px solid black`,
		    6: `border: 1px solid black`,
		    7: `border: 1px solid black`,
		    8: `border: 1px solid black`,
		    9: `border: 1px solid black`,
		    10: `border: 1px solid black`,
		    11: `border: 1px solid black`,
		    12: `border: 1px solid black`,
	    }
	  }
	, ascTop = 370
	, ascLeft = 240
	, descTop = -110
	, descLeft= 240
	, ascCols = Object.freeze([
      Array(2).fill(2),
		  Array(4).fill(3),
		  Array(6).fill(4),
		  Array(8).fill(5),
		  Array(10).fill(6),
		  Array(12).fill(7),
	  ])
	// -> [Array]
	, descCols = Object.freeze([
		  Array(10).fill(8),
		  Array(8).fill(9),
		  Array(6).fill(10),
		  Array(4).fill(11),
		  Array(2).fill(12),
	  ])
	, player1Chips = { ...defaultPlayerChips }
	, player2Chips = { ...defaultPlayerChips }
  , ropeLocations = Object.freeze([2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
		  .reduce((a, x, i) => ({
				...a, 
				[x]: 320 + (40*(i-1)),
			}), {}))
	
	// Int -> Int -> [Int, Int] -> Int -> Object
   , getRopeLocations = (rope, ropeLen, startingPosition, incrementAmount) => {

		   const ropeLocs = Array(ropeLen).fill(rope).reduce((a, x, i) => {
			   const nextPos = a.curPos - incrementAmount
			   return {
				   ...a,
				   [i]: [getX(x), nextPos],
				   curPos: nextPos,
			   }
		   }, {curPos: startingPosition})

		   return ropeLocs
	  }
	// -> Object
	, ropePositions = Object.freeze({
		  2: getRopeLocations(2, 3, 415, 37),
		  3: getRopeLocations(3, 5, 445, 37),
		  4: getRopeLocations(4, 7, 475, 37),
		  5: getRopeLocations(5, 9, 515, 37),
		  6: getRopeLocations(6, 11, 550, 37),
		  7: getRopeLocations(7, 13, 585, 37),
		  8: getRopeLocations(8, 11, 550, 37),
		  9: getRopeLocations(9, 9, 515, 37),
		  10: getRopeLocations(10, 7, 475, 37),
		  11: getRopeLocations(11, 5, 445, 37),
		  12: getRopeLocations(12, 3, 415, 37),
	  })
	;

	let startingPositions = { ...defaultPlayerChips }
	function moveChips(ropeChoice) {
		moveChip(ropeChoice[0])
		if (ropeChoice[1]) moveChip(ropeChoice[1])
		state.turnPhase = `roll`
		ps = []
	}
	function moveChip (rope) {
		const playerRopePos = startingPositions[rope]
		const chipOnRope = state.activeChips.find(chip => {
			if (chip.rope === rope) return chip
		})
		const availableChip = state.activeChips.find(x => x.isAvailable)
		const chipToUse = chipOnRope || availableChip
		const playerChips = state.player === 1 ? state.player1Chips : state.player2Chips
		const playerPosition = playerChips[rope]
		const position = (chipToUse.position || playerPosition) + 1 
		const location = getNewLocation(rope, playerRopePos)
	  const updatedChip = {
			...chipToUse,
			location,
			isAvailable: false,
			rope,
			position,
	  }

		state.activeChips = state.activeChips.map(x => {
			if (x.id === updatedChip.id) return updatedChip
			return x
		})	

		startingPositions[rope] = startingPositions[rope]+1
	}
  function getNewLocation (rope, playerPos) {
		return ropePositions[rope][playerPos]
	}
	function playerGetsRope (rope) {
		if (state.player === 1) {
			colColors[rope] = `blue`
			state.Player1Ropes = state.player1Ropes.concat(rope)
		}
		else if (state.player === 2) {
			colColors[rope] = `red`
			state.Player2Ropes = state.player2Ropes.concat(rope)
		}
	}
	function getX (rope) {
		return ropeLocations[rope]
	}
	function roleDice () {
    ps = [] // reset
		const role = () => Math.floor(Math.random()*6+1)
		die1 = role()
		die2 = role()
		die3 = role()
		die4 = role()
		state.turnPhase = `choose`
		const combinations = [
			[
				Number(die1) + Number(die2), 
			  Number(die3) + Number(die4),
			],
			[
				Number(die1) + Number(die3),
				Number(die2) + Number(die4),
			],
			[
				Number(die1) + Number(die4),
				Number(die2) + Number(die3),
			]
		]
		
		ps = combinations.reduce((a, [x1, x2]) => {
			if (playerHasActiveChipOnNumber(x1) && playerHasActiveChipOnNumber(x2)) 
				return [...a, [x1, x2]]
			if (playerHasTwoAvailableChips(x1, x2)) return [...a, [x1, x2]]
			if (playerHasAvailableChip(x1) && playerHasActiveChipOnNumber(x2)) 
				return [...a, [x1, x2]]
			if (playerHasAvailableChip(x2) && playerHasActiveChipOnNumber(x1)) 
				return [...a, [x1, x2]]
			if (playerHasAvailableChip(x1)) return [...a, [x1]]
			if (playerHasAvailableChip(x2)) return [...a, [x2]]
			if (playerHasActiveChipOnNumber(x1)) return [...a, [x1]]
			if (playerHasActiveChipOnNumber(x2)) return [...a, [x2]]
			/*if (playerHasActiveChipOnNumber(x1) && playerHasActiveChipOnNumber(x2)) 
				return [...a, [x1, x2]]
			if (playerHasAvailableChip(x1, x2)) return [...a, [x1], [x2]]
			if (playerHasActiveChipOnNumber(x1)) return [...a, [x1]]
			if (playerHasActiveChipOnNumber(x2)) return [...a, [x2]]*/
			return a
		}, [])
		if (!ps.length) {
	    alert(`lost turn!`)
			reset()
		}
	}
	function ropeIsPlayable (rope) {
		return !state.ownedRopes.includes(rope) && chipIsNotAtTop(rope)
	}
	function playerHasTwoAvailableChips (rope1, rope2) {
		return state.activeChips.filter(x => x.isAvailable).length >= 2 
			&& ropeIsPlayable(rope1) && ropeIsPlayable(rope2)
	}
	function playerHasActiveChipOnNumber (rope) {
		return !!state.activeChips.find(x => x.rope === rope)
		&& (!state.player1Ropes.includes(rope) && !state.player2Ropes.includes(rope))
		&& chipIsNotAtTop(rope)
	}
	function chipIsNotAtTop (rope) {
		const chip = state.activeChips.find(x => x.rope === rope)
		if (!chip) return true
		const maxPosition = Object.keys(ropePositions[rope]).length
		return chip.position+1 !== maxPosition
	}
	function playerHasAvailableChip (rope) {
		return !!state.activeChips.find(x => x.isAvailable) && ropeIsPlayable(rope)
	}
	function reset () {
		state.player = state.player === 1 ? 2 : 1
		state.activeChips = [ ...defaultChips ]
		state.turnPhase = `roll`
		if (state.player === 1) {
			startingPositions = { ...state.player1Chips }
		} else {
			startingPositions = { ...state.player2Chips }
		}
	}
	function lockInGains () {
    updatePlayerPosition()
		reset()
	}
	function updatePlayerPosition () {
		state.activeChips.forEach(chip => {
			const maxPosition = Object.keys(ropePositions[chip.rope]).length
			if (state.player === 1) {
				state.player1Chips[chip.rope] = chip.position
				console.log(chip.position, maxPosition)
				if (chip.position+1 === maxPosition) {
					state.colColors[chip.rope] = 'background-color: blue'
					state.ownedRopes = state.ownedRopes.concat(chip.rope)
				}
			} else {
				state.player2Chips[chip.rope] = chip.position
				console.log(chip.position, maxPosition)
				if (chip.position+1 === maxPosition) {
					state.colColors[chip.rope] = 'background-color: red'
					state.ownedRopes = state.ownedRopes.concat(chip.rope)
				}
			}
		})
	}
</script>
<style>
	.border {
		border: 4px solid black;
		width: 370px;
		height: 370px;
		transform: rotate(45deg);
		position: absolute;
		left: 299px;
		top: 170px
	}
	.col-c {
		position: absolute;
	}
	.col {
		display: flex;
		flex-direction: column;
	}
	.active-chip {
		position: absolute;
		height: 20px;
		width: 20px;
		background-color: black;
		border-radius: 50%;
	}
	.player1-chip {
		position: absolute;
		height: 20px;
		width: 20px;
		background-color: blue;
		border-radius: 50%;
	}
	.player2-chip {
		position: absolute;
		height: 20px;
		width: 20px;
		background-color: red;
		border-radius: 50%;
	}
	.btn-container {
		position: absolute;
		right: 50px;
	}
	.debug {
		position: absolute;
		bottom: 50px;
	}
</style>
