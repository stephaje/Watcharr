<script lang="ts">
  import { watchedList } from "@/store";
  import PageError from "@/lib/PageError.svelte";
  import Spinner from "@/lib/Spinner.svelte";
  import axios from "axios";
  import type { TMDBDiscoverMovies, TMDBUpcomingMovies } from "@/types";
  import Poster from "@/lib/poster/Poster.svelte";
  import { getWatchedDependedProps } from "@/lib/util/helpers";
  import PosterList from "@/lib/poster/PosterList.svelte";

  $: wList = $watchedList;
  $: wIds = [...wList.map((i) => i.content?.tmdbId)];

  // TODO, auto fetch more as the lists get smaller
  let trendingPage = 1;
  let upcomingPage = 1;

  async function trendingMovies(page: number) {
    return (await axios.get(`/content/discover/movies/` + page)).data as TMDBDiscoverMovies;
  }

  async function upcomingMovies(page: number) {
    return (await axios.get(`/content/upcoming/movies/` + page)).data as TMDBUpcomingMovies;
  }
</script>

<svelte:head>
  <title>Discover Content</title>
</svelte:head>

<div class="page">
  <h1>Discover</h1>

  <h2 class="norm">Trending Movies</h2>
  {#await trendingMovies(trendingPage)}
    <Spinner />
  {:then movies}
    <PosterList>
      {#each movies.results as movie}
        {#if wIds.indexOf(movie.id) == -1}
          <Poster
            media={{ ...movie, media_type: "movie" }}
            {...getWatchedDependedProps(movie.id, "movie", wList)}
            small={true}
          />
        {/if}
      {/each}
    </PosterList>
  {:catch err}
    <PageError pretty="Failed to load discovered movies!" error={err} />
  {/await}

  <h2 class="norm">Upcoming Movies</h2>
  {#await upcomingMovies(upcomingPage)}
    <Spinner />
  {:then shows}
    <PosterList>
      {#each shows.results as tv}
        {#if wIds.indexOf(tv.id) == -1}
          <Poster
            media={{ ...tv, media_type: "movie" }}
            {...getWatchedDependedProps(tv.id, "movie", wList)}
            small={true}
          />
        {/if}
      {/each}
    </PosterList>
  {:catch err}
    <PageError pretty="Failed to load upcoming movies!" error={err} />
  {/await}
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
