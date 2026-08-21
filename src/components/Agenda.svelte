<script>
  import { onMount } from 'svelte';
  import SvelteMarkdown from 'svelte-markdown';
  import { DateTime } from 'luxon';
  import simplur from 'simplur';

  let loading = true;

  let agenda_list = [];
  let categories = [];
  let venues = [];

  let active_tab = '';
  let active_venue = 'TIZ Kochi';

  async function loadData() {
    const today = DateTime.now().toFormat('LLL dd, yyyy');

    loading = true;

    try {
      const response = await fetch('https://events.startupmission.in/api/event/mirror/agenda/venue');
      const data = await response.json();

      agenda_list = data.agenda ?? {};
      categories = data.categories ?? [];
      venues = data.venues ?? [];

      const availableDates = dates();

      active_tab = availableDates.includes(today) ? today : availableDates[0] ?? '';

      active_venue = 'TIZ Kochi';
    } catch (error) {
      console.error('Failed to load agenda:', error);
    } finally {
      loading = false;
    }
  }

  function dates() {
    return Object.keys(agenda_list);
  }

  onMount(async () => {
    await loadData();
  });

  function date_format(date, format) {
    const d = DateTime.fromFormat(date, 'LLL dd, yyyy');
    return d.toFormat(format);
  }

  function date_single(date) {
    const d = DateTime.fromSQL(date);
    return d.toFormat('hh:mm a');
  }

  function date_detail(start, end) {
    const s = DateTime.fromSQL(start);
    const e = DateTime.fromSQL(end);

    return `${s.toFormat('hh:mm a')} - ${e.toFormat('hh:mm a')}`;
  }

  function selectTab(date) {
    active_tab = date;
  }
</script>

{#if loading}

  <div class="text-center text-[#ffffff] py-10">
    Loading schedule...
  </div>

{:else}

  <!-- DATE TABS -->
  <div class="agenda__tabs space-x-4 text-center text-[#ffffff] mb-5">
    {#each Object.entries(agenda_list) as [date, agendas]}
      <button on:click={() => selectTab(date)} class="px-5 py-3 spl_cursor mb-3 sm:text-base text-xs rounded-md shadow border transition-all duration-200 {date === active_tab ? 'bg-[#ff6a3d] text-[#ffffff] border-[#ff6a3d]' : 'border-white'}">
        <span>{date_format(date, 'ccc')}</span>, {date_format(date, 'LLL d, y')}
      </button>
    {/each}
  </div>

  <!-- EVENT SCHEDULE -->
  <div id="event-sched1" data-title="Event Schedule">

    {#each Object.entries(agenda_list) as [date, agendas]}

      {#if date === active_tab}

        <div class="w-full max-w-6xl mx-auto mb-5">

          {#each Object.entries(agendas) as [venue, agenda]}

            {#if venue === active_venue}

              {#each agenda as {name, description, start_time, end_time, category, speakers}}

                <div class="alternate-row px-5 py-3">

                  <div class="flex flex-col md:flex-row md:space-x-5">

                    <div class="text-[#ffffff] md:pt-3 flex-shrink-0 min-w-[6rem]">
                      <p class="text-[#ffffff] text-[14px]">{date_single(start_time)} IST</p>
                    </div>

                    <div class="w-full divide-y divide-gray-600 space-y-4 mb-3">

                      <div class="w-full pt-3">

                        {#if category}
                          <div class="uppercase">
                            <p class="text-[#ffffff] text-[13px]">{category}</p>
                          </div>
                        {/if}

                        <div class="text-[15px] font-bold">
                          <p class="text-[#ffffff] uppercase">{name}</p>
                        </div>

                        {#if description}
                          <div class="markdown-content mt-4 text-[#7d7a7a] mb-4">
                            <SvelteMarkdown source={description} />
                          </div>
                        {/if}

                        {#if speakers && Object.keys(speakers).length > 0}

                          <div class="mt-4 ml-4">

                            {#each Object.entries(speakers) as [speakerCategory, speaker_list]}

                              {#if speaker_list?.length}

                                <div class="font-bold uppercase text-pink-300 mb-3">
                                  {simplur`${[speaker_list.length, null]}${speakerCategory}[|s]`}
                                </div>

                                <div class="flex flex-wrap w-full">

                                  {#each speaker_list as {name, designation, organisation, photo, linkedin}}

                                    <div class="flex items-center space-x-4 mb-3 pr-3 md:w-1/2">

                                      {#if photo}
                                        <div class="rounded-full flex-shrink-0 w-20 h-20 border-r-2 border-b-2 border-pink-200 shadow-lg overflow-hidden">
                                          <img src={photo} alt={name} class="w-full h-full object-cover" />
                                        </div>
                                      {/if}

                                      <div>

                                        <div class="font-bold text-[#ffffff]">
                                          {#if linkedin}
                                            <a href={linkedin} target="_blank" rel="noopener noreferrer" class="hover:underline">
                                              {name}
                                            </a>
                                          {:else}
                                            {name}
                                          {/if}
                                        </div>

                                        {#if designation}
                                          <div class="text-[12px] text-pink-300">{designation}</div>
                                        {/if}

                                        {#if organisation}
                                          <div class="text-[12px] text-pink-300">{organisation}</div>
                                        {/if}

                                      </div>

                                    </div>

                                  {/each}

                                </div>

                              {/if}

                            {/each}

                          </div>

                        {/if}

                      </div>

                    </div>

                  </div>

                </div>

              {/each}

            {/if}

          {/each}

        </div>

      {/if}

    {/each}

  </div>

{/if}

<style lang="scss">
  .agenda__tabs {
    button {
      transition: background-color 0.2s ease, border-color 0.2s ease, transform 0.2s ease;

      &:hover {
        transform: translateY(-1px);
      }
    }
  }

  .alternate-row {
    border: 1px solid rgb(255 255 255 / 0.1);

    &:nth-child(odd) {
      background-color: rgb(255 255 255 / 0.1);
    }

    &:nth-child(even) {
      background-color: transparent;
    }
  }

  :global(.markdown-content h1) {
    @apply text-base font-bold text-[#d5d5d5] mb-4;
  }

  :global(.markdown-content h2) {
    @apply text-base font-bold text-[#d5d5d5] mb-4;
  }

  :global(.markdown-content h3) {
    @apply text-sm font-bold uppercase text-pink-300 mb-3;
  }

  :global(.markdown-content h4) {
    @apply text-sm font-bold text-[#d5d5d5] mb-3;
  }

  :global(.markdown-content p) {
    @apply text-sm text-[#d5d5d5] leading-6 mb-4;
  }

  :global(.markdown-content ul) {
    @apply list-disc pl-5 mb-4 space-y-1;
  }

  :global(.markdown-content ol) {
    @apply list-decimal pl-5 mb-4 space-y-1;
  }

  :global(.markdown-content li) {
    @apply text-sm text-[#d5d5d5] leading-6;
  }

  :global(.markdown-content strong) {
    @apply font-bold text-[#d5d5d5];
  }

  :global(.markdown-content a) {
    @apply underline text-pink-300;
  }

  #event-sched {
    position: relative;

    &::before {
      @apply text-[#ffffff] md:text-8xl text-5xl font-bold opacity-10 transform -rotate-90;

      content: attr(data-title);
      position: absolute;
      top: 0;
      left: -75%;
      transform-origin: center right;

      @media (min-width: 768px) {
        & {
          font-size: 8rem;
          line-height: 1;
        }
      }
    }
  }
</style>