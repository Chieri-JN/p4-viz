<script lang="ts">
    import dayData from '$lib/data.json';
    import DayCell from '$lib/components/DayCell.svelte';
    import SongCard from '$lib/components/SongCard.svelte';
    import type { DayData } from '$lib/types';

    import bar1 from '$lib/BPM Bars/Bar-75-95.svg';
    import bar2 from '$lib/BPM Bars/Bar-96-115.svg';
    import bar3 from '$lib/BPM Bars/Bar-116-135.svg';
    import bar4 from '$lib/BPM Bars/Bar-136-155.svg';
    import bar5 from '$lib/BPM Bars/Bar-156.svg';

    import shapeMellowed from '$lib/Cell Shapes/orange-01.svg';
    import shapeNeutral from '$lib/Cell Shapes/orange-02.svg';
    import shapeEnergized from '$lib/Cell Shapes/orange-03.svg';

    const BPM_LEGEND = [
        { label: '75–95',   img: bar1 },
        { label: '95–115',  img: bar2 },
        { label: '116–135', img: bar3 },
        { label: '136–155', img: bar4 },
        { label: '155+',    img: bar5 }
    ];

    const FEELING_LEGEND = [
        { label: 'Mellowed',  img: shapeMellowed },
        { label: 'Neutral',   img: shapeNeutral },
        { label: 'Energized', img: shapeEnergized }
    ];

    const DAY_NAMES = ['SUN', 'MON', 'TUE', 'WED', 'THU', 'FRI', 'SAT'];
    const MONTH_NAMES = [
        'January', 'February', 'March', 'April', 'May', 'June',
        'July', 'August', 'September', 'October', 'November', 'December'
    ];

    type Cell =
        | { kind: 'data'; data: DayData; day: number; month: number; isMonthStart: boolean }
        | { kind: 'pad' };

    const data = dayData as DayData[];

    // Sorted, continuous list of data days (no calendar gaps)
    const sortedData = [...data].sort((a, b) => a.date.localeCompare(b.date));

    // Build a single flat list of cells aligned to weekday columns.
    // - Lead with empty cells so the first data day sits in its real weekday column
    // - All days then flow continuously into each other across month boundaries
    // - No trailing empty cells
    const cells: Cell[] = (() => {
        const out: Cell[] = [];
        if (sortedData.length === 0) return out;

        const [fy, fm, fd] = sortedData[0].date.split('-').map(Number);
        const firstDow = new Date(fy, fm - 1, fd).getDay();
        for (let i = 0; i < firstDow; i++) out.push({ kind: 'pad' });

        let prevMonth = -1;
        for (const d of sortedData) {
            const [, m, day] = d.date.split('-').map(Number);
            out.push({
                kind: 'data',
                data: d,
                day,
                month: m,
                isMonthStart: m !== prevMonth
            });
            prevMonth = m;
        }
        return out;
    })();

    function dateRangeLabel(): string {
        if (sortedData.length === 0) return '';
        const fmt = (iso: string) => {
            const [, m, d] = iso.split('-').map(Number);
            return `${MONTH_NAMES[m - 1].slice(0, 3)} ${d}`;
        };
        return `${fmt(sortedData[0].date)} – ${fmt(sortedData[sortedData.length - 1].date)}`;
    }

    // Tooltip state
    let ttVisible = $state(false);
    let ttData = $state<DayData | null>(null);
    let ttX = $state(0);
    let ttY = $state(0);

    const TT_W = 320;
    const TT_H = 460;

    function placeTooltip(e: MouseEvent) {
        let x = e.clientX + 18;
        let y = e.clientY - TT_H / 2;
        if (x + TT_W > window.innerWidth - 8) x = e.clientX - TT_W - 18;
        if (x < 8) x = 8;
        if (y < 8) y = 8;
        if (y + TT_H > window.innerHeight - 8) y = window.innerHeight - TT_H - 8;
        ttX = x;
        ttY = y;
    }

    function showTooltip(e: MouseEvent, d: DayData) {
        ttData = d;
        ttVisible = true;
        placeTooltip(e);
    }
    function moveTooltip(e: MouseEvent, _d: DayData) {
        placeTooltip(e);
    }
    function hideTooltip() {
        ttVisible = false;
    }

    function formatDate(iso: string): string {
        const [y, m, d] = iso.split('-').map(Number);
        return `${MONTH_NAMES[m - 1]} ${d}, ${y}`;
    }
</script>

<div class="page">
    <header class="hero">
        <!-- <h1>A Month <em>in Music</em></h1> -->
         <h1>How the weather affected the music I listen to</h1>
        <p class="lede">
            Over the span of 30 days, I tended to listen to more energizing music when it was brighter outside and more mellowed music when its darker outside. 
            Contrary to my initial expectations, the BPM of the song did not have a significant correlation with my mood. 
            <!-- A daily soundtrack mapped to weather, mood, and tempo. Each
            <span class="dot orange"></span> sunny or
            <span class="dot blue"></span> cloudy day is a donut—the shape below
            reflects how the song made me feel, the bar across it shows its BPM. -->
            Hover any day to play the track.
        </p>
    </header>

    <div class="body">
        <aside class="legend">
            <div class="legend-block">
                <div class="legend-title">Weather</div>
                <div class="legend-item">
                    <span class="dot orange"></span> Sunny
                </div>
                <div class="legend-item">
                    <span class="dot blue"></span> Cloudy
                </div>
            </div>
            <div class="legend-block">
                <div class="legend-title">Song BPM</div>
                {#each BPM_LEGEND as { label, img }}
                    <div class="legend-item bpm-item">
                        <img class="bpm-swatch" src={img} alt="" />
                        <span>{label}</span>
                    </div>
                {/each}
            </div>
            <div class="legend-block">
                <div class="legend-title">How the song made me feel</div>
                {#each FEELING_LEGEND as { label, img }}
                    <div class="legend-item feeling-item">
                        <img class="feeling-swatch" src={img} alt="" />
                        <span>{label}</span>
                    </div>
                {/each}
            </div>
        </aside>

        <main class="calendar-col">
            <section class="month">
                <h2 class="month-title">
                    {dateRangeLabel()}
                    <span class="year">2026</span>
                </h2>

                <div class="day-header-row">
                    {#each DAY_NAMES as name}
                        <div class="day-header">{name}</div>
                    {/each}
                </div>

                <div class="calendar-grid">
                    {#each cells as cell}
                        {#if cell.kind === 'pad'}
                            <div class="empty-cell"></div>
                        {:else}
                            <div class="cell-wrap" class:month-start={cell.isMonthStart && cells.indexOf(cell) > 0}>
                                {#if cell.isMonthStart && cells.indexOf(cell) > 0}
                                    <span class="month-marker">
                                        {MONTH_NAMES[cell.month - 1].slice(0, 3)}
                                    </span>
                                {/if}
                                <DayCell
                                    dayData={cell.data}
                                    onhover={showTooltip}
                                    onmove={moveTooltip}
                                    onleave={hideTooltip}
                                />
                            </div>
                        {/if}
                    {/each}
                </div>
            </section>
        </main>
    </div>
</div>

{#if ttVisible && ttData}
    <div class="tooltip" style="left:{ttX}px; top:{ttY}px;">
        <div class="tooltip-header">
            <div class="tooltip-date">{formatDate(ttData.date)}</div>
            <div class="tooltip-meta">
                {ttData.weather.trim()} · {ttData.bpm} BPM · {ttData.feeling.trim()}
            </div>
        </div>
        <SongCard embedUrl={ttData.spotifyEmbed} />
        {#if ttData.notes}
            <div class="tooltip-notes">"{ttData.notes}"</div>
        {/if}
    </div>
{/if}

<style>
    .page {
        max-width: 1280px;
        margin: 0 auto;
        padding: 3rem 2rem 4rem;
        font-family: var(--font-leaguespartan);
        color: var(--black-sub);
    }

    .body {
        display: grid;
        grid-template-columns: 200px minmax(0, 1fr);
        gap: 3rem;
        align-items: start;
    }

    .calendar-col {
        min-width: 0;
    }

    .hero {
        margin-bottom: 3rem;
        max-width: 760px;
    }

    .hero h1 {
        font-family: var(--font-gabarito);
        font-size: clamp(2.4rem, 5vw, 4rem);
        font-weight: 800;
        line-height: 1.05;
        letter-spacing: -0.02em;
        margin: 0;
    }

    /* .hero h1 em {
        font-style: italic;
        color: var(--primary-color);
    } */

    .lede {
        margin-top: 1rem;
        font-size: 1rem;
        line-height: 1.55;
        color: var(--dark-grey-sub);
        opacity: 0.85;
    }

    .dot {
        display: inline-block;
        width: 0.7em;
        height: 0.7em;
        border-radius: 50%;
        vertical-align: middle;
        margin: 0 0.15em;
    }
    .dot.orange { background: var(--primary-color); }
    .dot.blue   { background: var(--secondary-color); }

    .month {
        margin-bottom: 3rem;
    }

    .month-title {
        font-family: var(--font-gabarito);
        font-size: 1.6rem;
        font-weight: 700;
        margin: 0 0 1rem;
        letter-spacing: -0.01em;
    }

    .month-title .year {
        font-weight: 400;
        color: var(--black-sub);
        opacity: 0.45;
        margin-left: 0.4em;
    }

    .day-header-row {
        display: grid;
        grid-template-columns: repeat(7, 1fr);
        gap: 7px;
        margin-bottom: 8px;
    }

    .day-header {
        font-family: var(--font-gabarito);
        font-size: 0.7rem;
        letter-spacing: 0.50em;
        text-align: center;
        color: var(--black-sub);
        opacity: 0.45;
        padding-bottom: 6px;
        border-bottom: 1px solid var(--grey-sub);
    }

    .calendar-grid {
        display: grid;
        grid-template-columns: repeat(7, 1fr);
        gap: 18px;
        align-items: start;
        justify-items: center;
    }

    .empty-cell {
        width: 100%;
        min-height: 130px;
    }

    .cell-wrap {
        position: relative;
        display: flex;
        justify-content: center;
        width: 100%;
        /* padding: 10px; */
    }

    .month-marker {
        position: absolute;
        top: -1.25rem;
        left: 50%;
        transform: translateX(-50%);
        font-family: var(--font-gabarito);
        font-size: 1rem;
        font-weight: 700;
        letter-spacing: 0.12em;
        text-transform: uppercase;
        color: var(--primary-color);
        white-space: nowrap;
    }

    .legend {
        position: sticky;
        top: 2rem;
        display: flex;
        flex-direction: column;
        gap: 1.75rem;
        padding: 1.25rem 1rem;
        border-right: 1px solid var(--grey-sub);
        font-size: 0.85rem;
    }

    .legend-block {
        display: flex;
        flex-direction: column;
    }

    .legend-title {
        font-family: var(--font-gabarito);
        font-weight: 700;
        letter-spacing: 0.04em;
        text-transform: uppercase;
        font-size: 0.72rem;
        margin-bottom: 0.5rem;
        color: var(--black-sub);
        opacity: 0.6;
    }

    .legend-item {
        padding: 2px 0;
        color: var(--dark-grey-sub);
        opacity: 0.8;
    }

    .bpm-item {
        display: flex;
        align-items: center;
        gap: 8px;
    }

    .bpm-swatch {
        width: 28px;
        height: 28px;
        object-fit: contain;
        flex-shrink: 0;
    }

    .feeling-item {
        display: flex;
        align-items: center;
        gap: 8px;
    }

    .feeling-swatch {
        width: 28px;
        height: 28px;
        object-fit: contain;
        flex-shrink: 0;
    }

    /* Tooltip */
    .tooltip {
        position: fixed;
        z-index: 1000;
        width: 320px;
        background: var(--white-sub);
        border: 1px solid var(--grey-sub);
        border-radius: 12px;
        padding: 12px;
        pointer-events: none;
        box-shadow: 0 12px 32px rgba(0, 0, 0, 0.12);
        font-family: var(--font-leaguespartan);
    }

    .tooltip-header {
        margin-bottom: 8px;
    }

    .tooltip-date {
        font-family: var(--font-gabarito);
        font-weight: 700;
        font-size: 0.95rem;
        color: var(--black-sub);
        letter-spacing: -0.01em;
    }

    .tooltip-meta {
        font-size: 0.72rem;
        letter-spacing: 0.08em;
        text-transform: uppercase;
        color: var(--black-sub);
        opacity: 0.55;
        margin-top: 2px;
    }

    .tooltip-notes {
        margin-top: 8px;
        font-size: 0.78rem;
        font-style: italic;
        color: var(--dark-grey-sub);
        opacity: 0.75;
        line-height: 1.4;
    }

    @media (max-width: 900px) {
        .body {
            grid-template-columns: 1fr;
            gap: 2rem;
        }
        .legend {
            position: static;
            border-right: none;
            border-bottom: 1px solid var(--grey-sub);
            padding: 0 0 1.5rem;
            flex-direction: row;
            flex-wrap: wrap;
            gap: 2rem 3rem;
        }
    }

    @media (max-width: 720px) {
        .day-header-row,
        .calendar-grid {
            gap: 6px;
        }
        .tooltip {
            width: 280px;
        }
    }
</style>
