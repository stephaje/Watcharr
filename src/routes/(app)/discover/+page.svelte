<script lang="ts">
  import { watchedList } from "@/store";
  import Spinner from "@/lib/Spinner.svelte";
  import axios from "axios";
  import type { TMDBDiscoverMovies, TMDBResult, TMDBUpcomingMovies } from "@/types";
  import Poster from "@/lib/poster/Poster.svelte";
  import { getWatchedDependedProps } from "@/lib/util/helpers";
  import PosterList from "@/lib/poster/PosterList.svelte";
  import { onMount } from "svelte";

  $: wList = $watchedList;
  $: wIds = [...wList.map((i) => i.content?.tmdbId)];

  let isLoaded = false;
  let isFetching = false;
  let trendingList: TMDBResult[] = [];
  $: trendingList = trendingList.filter((x) => wIds.indexOf(x.id) == -1);

  let trendingPage = 1;

  async function fetchTrendingMovies() {
    isFetching = true;
    let results = (await axios.get(`/content/discover/movies/` + trendingPage))
      .data as TMDBDiscoverMovies;
    trendingList.push(...results.results.filter((x) => wIds.indexOf(x.id) == -1));
    trendingList = trendingList;
    trendingPage++;
    isFetching = false;
  }

  onMount(async () => {
    await fetchTrendingMovies();
    setInterval(function () {
      if (trendingList.length <= 20 && isFetching == false) {
        fetchTrendingMovies();
      }
    }, 1000);
    isLoaded = true;
  });
</script>

<svelte:head>
  <title>Discover Content</title>
</svelte:head>

<div class="page">
  <h1>Discover</h1>

  <h2 class="norm">Trending Movies</h2>
  {#if !isLoaded}
    <Spinner />
  {:else}
    <PosterList>
      {#each trendingList as movie (movie.id)}
        <Poster
          media={{ ...movie, media_type: "movie" }}
          {...getWatchedDependedProps(movie.id, "movie", wList)}
          small={true}
        />
      {/each}
    </PosterList>
  {/if}
</div>

<style lang="scss">
  .page {
    display: flex;
    flex-flow: column;
    margin-left: auto;
    margin-right: auto;
    padding: 20px 50px;
    max-width: 1200px;

    h1 {
      margin-bottom: 15px;
    }

    h2 {
      font-variant: small-caps;
    }

    @media screen and (max-width: 500px) {
      padding: 20px;
    }
  }
</style>
