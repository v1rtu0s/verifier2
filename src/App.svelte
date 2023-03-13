<script>
  import { Button, Input, Field, Card, Container } from "svelte-chota";
  import { createHmac } from "crypto";
  import { tweened } from "svelte/motion";
  import { cubicOut } from "svelte/easing";

  let serverSeed = "";
  let clientSeed = "";
  let nonce = "";
  let activeTab = "limbo";
  let verify_bytes = [];
  let cursor = 0;
  let count = 20;	


  function generateBytes({ serverSeed, clientSeed, nonce, cursor, count }) {
    // Setup cursor variables
    let currentRound = Math.floor(cursor / 32);
    let currentRoundCursor = cursor;
    currentRoundCursor -= currentRound * 32;
    // Generate bytes until count is fulfilled
    const bytes = [];
    while (bytes.length < count) {
      // HMAC function used to output provided inputs into bytes
      const hmac = createHmac("sha256", serverSeed);
      hmac.update(`${clientSeed}:${nonce}:${currentRound}`);
      const buffer = hmac.digest();
      // Update cursor for next iteration of loop
      while (currentRoundCursor < 32 && bytes.length < count) {
        bytes.push(buffer[currentRoundCursor]);
        currentRoundCursor += 1;
      }
      currentRoundCursor = 0;
      currentRound += 1;
    }
    console.log(bytes);
    return bytes;
  }
  function generateFloats({ serverSeed, clientSeed, nonce, cursor, count }) {
    const bytes = generateBytes({
      serverSeed,
      clientSeed,
      nonce,
      cursor,
      count: count * 4
    });
    // Convert bytes to floats
    const floats = [];
    for (let i = 0; i < bytes.length; i += 4) {
      let partial = 0;
      for (let j = 0; j < 4; j++) {
        partial += bytes[i + j] / 256 ** (j + 1);
      }
      floats.push(partial);
    }
    console.log(floats);
    return floats;
  }


$: verify_bytes = generateBytes({ serverSeed, clientSeed, nonce, cursor, count });




	
</script>

<main>
	

	
<nav>
  
	<a class:active={activeTab === 'limbo'} href="#" on:click={() => activeTab = 'limbo'}>Limbo</a>
	  <a class:active={activeTab === 'dice'} href="#" on:click={() => activeTab = 'dice'}>Dice</a>
	  <a class:active={activeTab === 'diamonds'} href="#" on:click={() => activeTab = 'diamonds'}>Diamonds</a>
	  <a class:active={activeTab === 'wheel'} href="#" on:click={() => activeTab = 'wheel'}>Wheel</a>  
	</nav>
	
	<Container>
		<Card>
	  <input bind:value={serverSeed} placeholder="ServerSeed"/>
	<br>
	  <input bind:value={clientSeed} placeholder="ClientSeed"/>
	<br>
	  <input bind:value={nonce} placeholder="Nonce"/>
	<br>
	
	<div style="margin-left:1rem; max-width:300px; max-height:200px">
	
	<button class="button primary" >VERIFY</button>
	
	</div>
    </Card>
	</Container>
	
</main>

<style>
	 @import "https://unpkg.com/chota@latest";
  a {
    margin: 4px;
  }
  nav {
    margin-bottom: 1rem;
  }
  .active {
    color: #ff3d03;
    font-weight: 700;
  }
  .button {
    margin: 1rem;
  }
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>