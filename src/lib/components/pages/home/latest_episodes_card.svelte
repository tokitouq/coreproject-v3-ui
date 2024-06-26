<script lang="ts">
    import { page } from "$app/stores";
    import ImageLoader from "$components/shared/image/image_loader.svelte";
    import ScrollArea from "$components/shared/scroll_area.svelte";
    import { FormatDate } from "$functions/format_date";
    import { remove_slash_from_end } from "$functions/urls/remove_slash_at_end";
    import Play from "$icons/shapes/play.svelte";
    import { onMount } from "svelte";
    import { slide } from "svelte/transition";

    // Boolean flag to check if slide is last element

    export let anime: {
        id: number;
        cover: string;
        name: string;
        episode_number: number;
        release_date: string;
        synopsis: string;
    };

    // Formated anime details
    const formated_episode_number = String(anime.episode_number).padStart(2, "0");
    const formated_release_date = new FormatDate(anime.release_date).format_to_time_from_now;

    /* Bindings */
    let ANIMATION_DURATION = 300,
        visible_ratio: number;
    let scroll_area_element: HTMLElement, anime_episode: HTMLElement;
    let show_more_info = false,
        should_expand = false;

    onMount(() => {
        scroll_area_element = anime_episode?.parentElement?.parentElement?.parentElement! as HTMLElement;

        const observer = new IntersectionObserver((entries) => {
            entries.forEach((entry) => {
                visible_ratio = entry.intersectionRatio;
            });
        });
        observer.observe(anime_episode);
        return () => observer.unobserve(anime_episode);
    });

    /** Bindings */

    function handle_mouseenter() {
        if (visible_ratio < 0.8) should_expand = true;
        show_more_info = true;
    }

    function handle_mouseleave() {
        show_more_info = false;
        should_expand = false;
    }

    function handle_animationstart() {
        const parent_element = anime_episode.parentElement!;

        // Declare rects
        const parent_rect = parent_element.getBoundingClientRect(), // taking parent not scroll_area_element
            anime_episode_rect = anime_episode.getBoundingClientRect();

        const scroll_area_center = scroll_area_element.offsetHeight / 2;
        const anime_episode_center = anime_episode_rect.top - parent_rect.top + anime_episode_rect.height / 2;
        const target_scroll_top = anime_episode_center - scroll_area_center + parseInt(getComputedStyle(anime_episode.parentElement!).gap) || 0;

        scroll_area_element.scroll({
            top: target_scroll_top,
            behavior: "smooth"
        });
    }
</script>

<anime-episode
    bind:this={anime_episode}
    on:mouseenter={handle_mouseenter}
    on:mouseleave={handle_mouseleave}
    role="group"
    class="group relative h-[5vw] duration-300 ease-in-out hover:h-[16vw]"
>
    <ImageLoader
        src={anime.cover}
        class="absolute h-full w-full rounded-[0.75vw] object-cover object-center"
    />
    <gradient-overlay class="gradient absolute inset-0 bg-gradient-to-t from-surface-900/75 to-surface-900/0" />
    <gradient-overlay class="gradient absolute inset-0 bg-gradient-to-r from-surface-900/50 to-surface-900/0" />

    <episode-info class="absolute inset-0 flex items-start justify-between p-[1.3125vw]">
        <div class="flex flex-col gap-[0.25vw]">
            <episode-name class="text-[1vw] font-semibold leading-[1.1875vw] text-white">
                {anime.name}
            </episode-name>
            <episode-dates class="flex items-center gap-[0.35vw] text-[0.8vw] text-surface-50">
                <span class="font-semibold">
                    Ep {formated_episode_number}
                </span>
                <span>
                    aired {formated_release_date}
                </span>
            </episode-dates>
        </div>
        <a
            href="{remove_slash_from_end($page.url.pathname)}/mal/{anime.id}/episode/{anime.episode_number}"
            class="btn btn-icon h-[2.5vw] w-[2.5vw] rounded-full bg-warning-400 text-surface-900 transition-colors duration-300 group-hover:bg-white"
        >
            <Play class="w-[1.25vw]" />
        </a>
    </episode-info>

    {#if show_more_info}
        <more-anime-info
            in:slide={{ duration: ANIMATION_DURATION, delay: ANIMATION_DURATION * (2 / 3) }}
            out:slide={{ duration: ANIMATION_DURATION * (2 / 3) }}
            on:animationstart={handle_animationstart}
            class="absolute inset-x-0 bottom-0 flex flex-col gap-[0.5vw] p-[1.3125vw]"
        >
            <genres class="flex items-center md:my-[0.35vw] md:gap-[0.5vw]">
                {#each ["Action", "Romance", "Hentai"] as genre}
                    <genre class="bg-surface-50 font-semibold leading-none text-black md:rounded-[0.35vw] md:px-[0.6vw] md:py-[0.3vw] md:text-[0.8vw]">
                        {genre}
                    </genre>
                {/each}
            </genres>
            <ScrollArea
                offsetScrollbar
                class="h-[6vw] text-[0.8vw] leading-[1.15vw] text-surface-50"
            >
                {anime.synopsis}
            </ScrollArea>
        </more-anime-info>
    {/if}
</anime-episode>
