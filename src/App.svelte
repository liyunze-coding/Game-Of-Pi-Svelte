<script lang="ts">
  import GameOfPi from './lib/GameOfPi.svelte';
  import Modal from './lib/Modal.svelte';

	let showModal = true;
</script>

<main>
  <Modal bind:showModal>
    <div class="w-full">hello</div>
  </Modal>
  <GameOfPi />
</main>

<style>
  
</style>
