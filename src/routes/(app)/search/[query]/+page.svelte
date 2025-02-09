<script lang="ts">
  import Poster from "@/lib/poster/Poster.svelte";
  import PosterList from "@/lib/poster/PosterList.svelte";
  import { searchQuery, serverFeatures, watchedList } from "@/store";
  import PageError from "@/lib/PageError.svelte";
  import Spinner from "@/lib/Spinner.svelte";
  import axios from "axios";
  import { getWatchedDependedProps, getPlayedDependedProps } from "@/lib/util/helpers";
  import PersonPoster from "@/lib/poster/PersonPoster.svelte";
  import type {
    ContentSearch,
    ContentSearchMovie,
    ContentSearchPerson,
    ContentSearchTv,
    GameSearch,
    MediaType,
    PublicUser
  } from "@/types";
  import UsersList from "@/lib/UsersList.svelte";
  import { onDestroy, onMount } from "svelte";
  import Error from "@/lib/Error.svelte";
  import GamePoster from "@/lib/poster/GamePoster.svelte";
  import { get } from "svelte/store";
  import { notify } from "@/lib/util/notify.js";

  export let data;

  $: searchQ = $searchQuery;
  $: wList = $watchedList;

  type GameWithMediaType = GameSearch & { media_type: "game" };
  type CombinedResult =
    | ContentSearchMovie
    | ContentSearchTv
    | ContentSearchPerson
    | GameWithMediaType;

  async function search(query: string) {
    const f = get(serverFeatures);
    if (!f.games) {
      console.log("Search: Only for movies/tv");
      return (await axios.get<ContentSearch>(`/content/${query}`)).data.results;
    }
    console.log("Search: For movies/tv and games");
    // To get around promise.all rejecting both promises when one fails,
    // catch them separately and return empty object so we can still
    // display the other media types.
    const r = await Promise.all([
      axios.get<ContentSearch>(`/content/${query}`).catch((err) => {
        console.error("Movies/Tv search failed!", err);
        notify({ text: "Movie/Tv Search Failed!", type: "error" });
        return { data: { results: [] } };
      }),
      axios.get<GameSearch[]>(`/game/search/${query}`).catch((err) => {
        console.error("Game search failed!", err);
        notify({ text: "Game Search Failed!", type: "error" });
        return { data: [] };
      })
    ]);
    const games: GameWithMediaType[] = r[1].data.map((g) => ({
      ...g,
      media_type: "game"
    }));
    const d = new Array<CombinedResult>().concat
      .apply([], [r[0].data.results, games])
      ?.sort((a, b) => {
        let name = "";
        if (a.media_type === "game" || a.media_type === "tv" || a.media_type === "person") {
          name = a.name ?? "";
        } else if (a.media_type === "movie") {
          name = a.title ?? "";
        }

        let name2 = "";
        if (b.media_type === "game" || b.media_type === "tv" || b.media_type === "person") {
          name2 = b.name ?? "";
        } else if (b.media_type === "movie") {
          name2 = b.title ?? "";
        }

        if (name < name2) {
          return 1;
        }
        if (name > name2) {
          return -1;
        }
        return 0;
      });
    return d;
  }

  async function searchUsers(query: string) {
    return (await axios.get(`/user/search/${query}`)).data as PublicUser[];
  }

  onMount(() => {
    if (!searchQ && data.slug) {
      searchQuery.set(data.slug);
    }
  });

  onDestroy(() => {
    searchQuery.set("");
  });
</script>

<svelte:head>
  <title>Content Search</title>
</svelte:head>

<div class="content">
  <div class="inner">
    {#if data.slug}
      {#await searchUsers(data.slug) then results}
        {#if results?.length > 0}
          <UsersList users={results} />
        {/if}
      {:catch err}
        <PageError pretty="Failed to load users!" error={err} />
      {/await}

      {#await search(data.slug)}
        <Spinner />
      {:then results}
        <h2>Results</h2>
        <PosterList>
          {#if results?.length > 0}
            {#each results as w (w.id)}
              {#if w.media_type === "person"}
                <PersonPoster id={w.id} name={w.name} path={w.profile_path} />
              {:else if w.media_type === "game"}
                <GamePoster
                  media={{
                    id: w.id,
                    coverId: w.cover.image_id,
                    name: w.name,
                    summary: w.summary,
                    firstReleaseDate: w.first_release_date
                  }}
                  {...getPlayedDependedProps(w.id, wList)}
                />
              {:else}
                <Poster media={w} {...getWatchedDependedProps(w.id, w.media_type, wList)} />
              {/if}
            {/each}
          {:else}
            No Search Results!
          {/if}
        </PosterList>
      {:catch err}
        <Error pretty="Failed to load results!" error={err} />
      {/await}

      <!-- {#await searchGames(data.slug)}
        <Spinner />
      {:then results}
        <h2>Results</h2>
        <PosterList>
          {#if results?.length > 0}
            {#each results as w (w.id)}
              <GamePoster media={w} />
            {/each}
          {:else}
            No Search Results!
          {/if}
        </PosterList>
      {:catch err}
        <Error pretty="Failed to load results!" error={err} />
      {/await} -->

      <!-- {#await search(data.slug)}
        <Spinner />
      {:then results}
        <h2>Results</h2>
        <PosterList>
          {#if results?.results?.length > 0}
            {#each results.results as w (w.id)}
              {#if w.media_type === "person"}
                <PersonPoster id={w.id} name={w.name} path={w.profile_path} />
              {:else}
                <Poster media={w} {...getWatchedDependedProps(w.id, w.media_type, wList)} />
              {/if}
            {/each}
          {:else}
            No Search Results!
          {/if}
        </PosterList>
      {:catch err}
        <Error pretty="Failed to load results!" error={err} />
      {/await} -->
    {:else}
      <h2>No Search Query!</h2>
    {/if}
  </div>
</div>

<style lang="scss">
  .content {
    display: flex;
    width: 100%;
    justify-content: center;

    .inner {
      width: 100%;
      max-width: 1200px;

      h2 {
        margin-left: 15px;
      }
    }
  }
</style>
