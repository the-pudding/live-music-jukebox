<script>
  import { song } from "../stores/jukebox.js";
  import { globalScale } from "../stores/jukebox.js";
  import { onMount } from "svelte";
  import { orderby } from "../stores/jukebox.js";
  import { show_duplicates } from "../stores/jukebox.js";
  import { active_artist } from "../stores/jukebox.js";
  import { ready } from "../stores/jukebox.js";
  import { createEventDispatcher } from "svelte";
  import { csv } from "d3-fetch";
  import { group, descending, ascending, extent } from "d3-array";
  import Dial from "./Dial.svelte";
  import { scaleLinear } from "d3-scale";
  import artistDictionary from "../data/artist_dictionary.csv";

  const dispatch = createEventDispatcher();
  let active;
  let mounted = false;
  let trackDictionary;
  let flat_data = [];
  let grouped = [];
  let active_track_key;
  let filename;
  let variable_name;
  const diff_string = "difference_";
  let artist_songlist;
  let active_artist_name;
  let active_artist_list;
  const folder_name = "assets/data/final_data_0128/single_rows/";
  const endpoint = ".csv";

  let current_metric = "difference_overall";
  $: current_metric = diff_string.concat($orderby);

  let extent_values;
  let scale_extent;
  let scale;

  function updateSong(v) {
    dispatch("message", {
      text: v,
      active_artist_list,
      scale,
    });
  }

  onMount(() => {
    csv("assets/data/final_data_0128/track_name_dictionary.csv")
      .then((tracks) => {
        trackDictionary = tracks;
      })
      .then(() => {
        csv("assets/data/final_data_0128/songlist_wide_optimized.csv").then(
          (raw) => {
            flat_data = raw;

            updateScale();
            grouped = Array.from(
              group(flat_data, (d) => d.artist_id),
              ([artist_id, artist_songlist]) => ({
                artist_id,
                artist_name_studio: artistDictionary.find(
                  (a) => a.artist_id == artist_id
                )["artist_name_studio"],

                artist_songlist: artist_songlist.map((d) => ({
                  ...d,
                  difference_valence: +d.difference_valence,
                  difference_energy: +d.difference_energy,
                  difference_acousticness: +d.difference_acousticness,
                  difference_duration: +d.difference_duration,
                  difference_dance: +d.difference_dance,
                  difference_instrumentalness: +d.difference_instrumentalness,
                  difference_liveness: +d.difference_liveness,
                  // track_name_id,
                  track_name_studio: trackDictionary.find(
                    (a) => a.track_name_id == d.track_name_id
                  )["track_name_studio"],
                })),
              })
            );

            sortData($orderby);
            active_track_key = "18GiV1BaXzPVYpp9rmOg0E2Xc1Xd7q4bunmnYkwIwJGY";
            active_artist_name = grouped[0].artist_name_studio;
            filename = folder_name.concat(active_track_key, endpoint);

            csv(filename)
              .then((selected) => {
                $song = selected[0];
                $song["difference_scaled"] = scale($song[current_metric]);
              })
              .then(() => {
                active_artist_list = grouped.find(
                  (d) => d.artist_name_studio == $song.artist_name_studio
                );
              })
              .then(() => {
                updateSong($song);
                $ready = true;

                mounted = true;
              })
              .catch((error) => {
                console.log(error);
              });
          }
        );
      })
      .catch((error) => {
        console.log(error);
      });
  });

  function onSelect(d) {
    active_track_key = d.track_key;

    filename = folder_name.concat(active_track_key, endpoint);
    csv(filename)
      .then((selected) => {
        $song = selected[0];
        $song["difference_scaled"] = scale($song[current_metric]);
      })
      .then(() => {
        active_artist_list = grouped.find(
          (d) =>
            d.artist_id ==
            artistDictionary.find(
              (g) => g.artist_name_studio == $song.artist_name_studio
            )["artist_id"]
        );
      })
      .then(() => {
        updateSong($song);
      })
      .catch((error) => {
        console.log(error);
      });
  }

  function onArtistChange() {
    if (mounted) {
      active_artist_list = grouped.find(
        (d) =>
          d.artist_id ==
          artistDictionary.find((g) => g.artist_name_studio == $active_artist)[
            "artist_id"
          ]
      );
      active_track_key = active_artist_list.artist_songlist[0].track_key;
      filename = folder_name.concat(active_track_key, endpoint);

      csv(filename)
        .then((selected) => {
          $song = selected[0];
          $song["difference_scaled"] = scale($song[current_metric]);
        })
        .then(() => {
          updateSong($song);
          scrollToArtist(
            grouped.indexOf(
              grouped.find((v) => v.artist_name_studio == $active_artist)
            )
          );
        })
        .catch((error) => {
          console.log(error);
        });
    }
  }

  let scrollToArtist = (number_of_artists_ahead) => {
    if (mounted) {
      let x = document.querySelector("li.artist-name");
      let objectHeight = x.offsetHeight;
      let target = document.getElementsByClassName("artist-name selected")[0];
      target.parentNode.scrollTop = objectHeight * number_of_artists_ahead;
    }
  };

  $: $active_artist, onArtistChange();
  // handleNewArtist($active_artist);

  function sortData(v) {
    //reorder the data using v as column name
    grouped.forEach((g) => {
      artist_songlist = g.artist_songlist;

      artist_songlist.sort((a, b) =>
        descending(Math.abs(a[current_metric]), Math.abs(b[current_metric]))
      );
    });

    updateScale();
    grouped = grouped;
  }

  function updateScale() {
    extent_values = [];
    flat_data.forEach((d) => extent_values.push(Math.abs(d[current_metric])));

    scale_extent = extent(extent_values);

    scale = scaleLinear(scale_extent, [0, 1]);
    $globalScale = scale;
    flat_data.forEach(
      (d) => (d["difference_scaled"] = scale(Math.abs(d[current_metric])))
    );
    flat_data = flat_data;
  }

  let visible_track_keys = [];
  const identifier = "#";

  let handleNewArtist = (d) => {
    if ($active_artist != d) {
      $active_artist = d;

      var x = grouped.find((v) => v.artist_name_studio == $active_artist);

      scrollToArtist(
        grouped.indexOf(
          grouped.find((v) => v.artist_name_studio == $active_artist)
        )
      );
    }
  };

  function getUniqueList(songlist) {
    if ($show_duplicates) {
      let truncatedSonglist = [];
      let trackNames = [];
      songlist.forEach((d) => {
        if (trackNames.indexOf(d.track_name_studio) === -1) {
          truncatedSonglist.push(d);
          trackNames.push(d.track_name_studio);
        }
      });
      return truncatedSonglist;
    } else {
      return songlist;
    }
  }

  let current = "Bob Dylan";
  $: sortData($orderby);
</script>

{#if grouped.length}
  <ul>
    {#each grouped as d}
      <li
        class="artist-name"
        class:selected="{$active_artist === d.artist_name_studio}"
        on:click="{handleNewArtist(d.artist_name_studio)}">
        {d.artist_name_studio}
      </li>
      <div class="track-list">
        {#each getUniqueList(d.artist_songlist) as v}
          <li
            class:active="{active === v.track_key}"
            class:selected="{$active_artist === d.artist_name_studio}"
            id="{v.track_key}"
            class="track-name"
            on:click="{() => onSelect(v)}">
            <span>{v.track_name_studio}</span>
            <Dial value="{v['difference_scaled']}" />
          </li>
        {/each}
      </div>
    {/each}
  </ul>
{:else}
  <p>Loading data...</p>
{/if}

<style>
  p {
    font-family: var(--sans);
  }
  ul {
    height: 92vh;
    overflow-y: scroll;
    width: 100%;
    overflow-x: hidden;
  }

  li.active {
    background: red;
    color: var(--off-white);
  }

  li {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
  }

  li span {
    width: calc(100% - 3rem);
  }

  li.selected {
    display: flex;
  }

  .artist-name {
    color: #3c3332;
    font-weight: bold;
    font-family: var(--narrow);
    padding: 0.25rem 0 0.25rem 0.25rem;
    font-size: 1.25em;
    text-transform: uppercase;
    position: sticky;
    cursor: pointer;
    max-width: 100%;
    overflow: hidden;
    white-space: nowrap;
  }

  .artist-name:hover,
  .artist-name.selected {
    color: var(--red);
    background-color: var(--jukebox-med-tan);
  }

  .track-name {
    padding: 0 0.5rem 0 1rem;
    color: var(--off-black);
    font-size: 1em;
    font-family: var(--narrow);
    border-bottom: 1px solid var(--jukebox-med-tan);
    padding-bottom: 0.5rem;
    padding-top: 0.5rem;
    display: none;
    cursor: pointer;
  }

  .track-name:hover {
    background: var(--jukebox-med-tan);
    font-weight: 700;
  }

  .track-list-wrapper {
    display: none;
  }

  .track-list-wrapper .entered {
    display: block;
  }

  .header {
    background-color: var(--jukebox-brown);
    border-radius: 5px;
    color: var(--jukebox-light-tan);
    text-transform: capitalize;
    text-align: center;
    width: fit-content;
    right: 50%;
    margin-left: 25%;
    font-size: 2rem;
    padding: 0.5rem;
    margin-top: 1rem;
  }

  .track-list li:last-of-type {
    margin: 0 0 3rem 0;
  }

  @media only screen and (max-width: 1000px) {
    p {
      font-size: 0.8rem;
    }

    .artist-name {
      font-size: 1rem;
      padding: 0.125rem 0 0.125rem 0.25rem;
    }
  }
</style>
