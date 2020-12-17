<script>
  import { song } from "../stores/jukebox.js";
  import Player from "./Player.svelte";
  import Table from "./Table.svelte";
  import Searchbar from "./Searchbar.svelte";
  import Order from "./Order.svelte";
  import Songlist from "./Songlist.svelte";
  import Namecard from "./Namecard.svelte";

  let data = [];
  let search_string;

  function handleMessage(event) {
    search_string = event.detail.text.toLowerCase();
    console.log(search_string);
    data = data.filter((d) =>
      d.artist_name_studio.toLowerCase().includes({ search_string })
    );
  }

  // function updateList(event) {
  //   console.log(event.detail.text);
  //   console.log($song);
  // }
</script>

<section class="jukebox">
  <Searchbar on:message="{handleMessage}" />

  <div>
    <Songlist search_query="{search_string}" />
  </div>

  {#if $song}
    <section class="top">
      <div class="player">
        <Player
          preview="{$song.track_preview_url_studio}"
          art="{$song.studio_art_large}"
          label="Studio" />
      </div>
      <div class="namecard">
        <Namecard />
      </div>
      <div class="player">
        <Player
          preview="{$song.track_preview_url_live}"
          art="{$song.live_art_large}"
          label="Live" />
      </div>
    </section>

    <Table />
  {/if}
  <Order />
</section>

<style>
  .top {
    display: flex;
    max-width: 60em;
    margin: 0 auto;
    background-color: #fefbf7;
  }

  .player {
    border: 4px solid pink;
  }

  .jukebox {
    background-color: #bfb2a2;
  }
</style>