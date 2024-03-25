<script lang="ts">
  import Poster from "@/lib/poster/Poster.svelte";
  import PosterList from "@/lib/poster/PosterList.svelte";
  import { activeSort } from "@/store";
  import type { Watched } from "@/types";

  export let list: Watched[];
  export let isPublicList: boolean = false;

  $: sort = $activeSort;
  $: watched = list.sort((a, b) => sortList(a, b));
  $: onlyWatched = watched.filter((i) => i.rating && i.rating > 0 && !i.hotOrNot);
  $: onlyHot = watched.filter((i) => i.hotOrNot && i.hotOrNot == "hot");
  $: onlyNot = watched.filter((i) => i.hotOrNot && i.hotOrNot == "not");

  $: sort, sortAllLists();

  function sortAllLists() {
    watched = watched.sort((a, b) => sortList(a, b));
  }

  function sortList(a: Watched, b: Watched) {
    if (sort[0] === "DATEADDED" && sort[1] === "UP") {
      return Date.parse(a.createdAt) - Date.parse(b.createdAt);
    } else if (sort[0] === "ALPHA") {
      const atitle = a.content ? a.content.title : a.game ? a.game.name : "";
      const btitle = b.content ? b.content.title : b.game ? b.game.name : "";
      if (sort[1] === "UP") {
        return atitle.localeCompare(btitle);
      } else if (sort[1] === "DOWN") {
        return btitle.localeCompare(atitle);
      }
    } else if (sort[0] === "LASTCHANGED") {
      if (sort[1] === "UP") return Date.parse(a.updatedAt) - Date.parse(b.updatedAt);
      else if (sort[1] === "DOWN") return Date.parse(b.updatedAt) - Date.parse(a.updatedAt);
    } else if (sort[0] === "RATING") {
      if (sort[1] === "UP") return (a.rating ?? 0) - (b.rating ?? 0);
      else if (sort[1] === "DOWN") return (b.rating ?? 0) - (a.rating ?? 0);
    }
    // default DATEADDED DOWN
    return Date.parse(b.createdAt) - Date.parse(a.createdAt);
  }
</script>

{#if onlyWatched?.length > 0}
  <PosterList label="Unrated">
    {#each onlyWatched as w (w.id)}
      {#if w.content}
        <Poster
          id={w.id}
          media={{
            id: w.content.tmdbId,
            poster_path: w.content.poster_path,
            title: w.content.title,
            overview: w.content.overview,
            media_type: w.content.type,
            release_date: w.content.release_date,
            first_air_date: w.content.first_air_date
          }}
          rating={w.rating}
          status={w.status}
          hotOrNot={w.hotOrNot}
          extraDetails={{
            dateAdded: w.createdAt,
            dateModified: w.updatedAt,
            lastWatched: ""
          }}
        />
      {/if}
    {/each}
  </PosterList>
{/if}

{#if onlyHot?.length > 0}
  <PosterList label="Hot">
    {#each onlyHot as w (w.id)}
      {#if w.content}
        <Poster
          id={w.id}
          media={{
            id: w.content.tmdbId,
            poster_path: w.content.poster_path,
            title: w.content.title,
            overview: w.content.overview,
            media_type: w.content.type,
            release_date: w.content.release_date,
            first_air_date: w.content.first_air_date
          }}
          rating={w.rating}
          status={w.status}
          hotOrNot={w.hotOrNot}
          extraDetails={{
            dateAdded: w.createdAt,
            dateModified: w.updatedAt,
            lastWatched: ""
          }}
        />
      {/if}
    {/each}
  </PosterList>
{/if}

{#if onlyNot?.length > 0}
  <PosterList label="Not">
    {#each onlyNot as w (w.id)}
      {#if w.content}
        <Poster
          id={w.id}
          media={{
            id: w.content.tmdbId,
            poster_path: w.content.poster_path,
            title: w.content.title,
            overview: w.content.overview,
            media_type: w.content.type,
            release_date: w.content.release_date,
            first_air_date: w.content.first_air_date
          }}
          rating={w.rating}
          status={w.status}
          hotOrNot={w.hotOrNot}
          extraDetails={{
            dateAdded: w.createdAt,
            dateModified: w.updatedAt,
            lastWatched: ""
          }}
        />
      {/if}
    {/each}
  </PosterList>
{/if}

<style lang="scss">
  .empty-list {
    display: flex;
    flex-flow: column;
    gap: 5px;
    align-items: center;

    h2 {
      margin-top: 10px;
      align-self: center;
    }

    h4 {
      font-weight: normal;
    }

    button {
      width: max-content;
      padding-left: 20px;
      padding-right: 20px;
      margin-top: 15px;
    }
  }
</style>
