<script lang="ts">
    import type { DayData } from '$lib/types';
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    import orange01 from '$lib/Cell Shapes/orange-01.svg';
    import orange02 from '$lib/Cell Shapes/orange-02.svg';
    import orange03 from '$lib/Cell Shapes/orange-03.svg';
    import blue01 from '$lib/Cell Shapes/blue-01.svg';
    import blue02 from '$lib/Cell Shapes/blue-02.svg';
    import blue03 from '$lib/Cell Shapes/blue-03.svg';

    import bar1 from '$lib/BPM Bars/Bar-75-95.svg';
    import bar2 from '$lib/BPM Bars/Bar-96-115.svg';
    import bar3 from '$lib/BPM Bars/Bar-116-135.svg';
    import bar4 from '$lib/BPM Bars/Bar-136-155.svg';
    import bar5 from '$lib/BPM Bars/Bar-156.svg';

    type HoverFn = (e: MouseEvent, d: DayData) => void;
    type LeaveFn = () => void;

    let {
        dayData,
        size = 96,
        onhover,
        onmove,
        onleave
    }: {
        dayData: DayData;
        size?: number;
        onhover?: HoverFn;
        onmove?: HoverFn;
        onleave?: LeaveFn;
    } = $props();

    let svgEl: SVGSVGElement;

    function normWeather(w: string): 'Sunny' | 'Cloudy' {
        const t = (w ?? '').trim().toLowerCase();
        if (t.startsWith('sun') || t.startsWith('clear')) return 'Sunny';
        return 'Cloudy';
    }

    function normFeeling(f: string): 'Mellowed' | 'Neutral' | 'Energized' {
        const t = (f ?? '').trim().toLowerCase();
        if (t.startsWith('mell')) return 'Mellowed';
        if (t.startsWith('ener') || t.startsWith('energ')) return 'Energized';
        return 'Neutral';
    }

    function bpmBar(bpm: number): string {
        if (bpm >= 156) return bar5;
        if (bpm >= 136) return bar4;
        if (bpm >= 116) return bar3;
        if (bpm >= 95) return bar2;
        return bar1;
    }

    const cellShapeMap = {
        Sunny: { Mellowed: orange01, Neutral: orange02, Energized: orange03 },
        Cloudy: { Mellowed: blue01, Neutral: blue02, Energized: blue03 }
    } as const;

    const weather = $derived(normWeather(dayData.weather));
    const feeling = $derived(normFeeling(dayData.feeling));
    const isSunny = $derived(weather === 'Sunny');
    const ringColor = $derived(
        isSunny ? 'var(--primary-color)' : 'var(--secondary-color)'
    );
    const cellShape = $derived(cellShapeMap[weather][feeling]);
    const bpmImg = $derived(bpmBar(dayData.bpm));

    const dayNum = $derived(parseInt(dayData.date.slice(8, 10), 10));

    function drawDonut() {
        if (!svgEl) return;
        const svg = d3.select(svgEl);
        svg.selectAll('*').remove();

        const r = size / 2;
        const outerR = r * 0.92;
        const innerR = r * 0.58;
        const ringWidth = outerR - innerR;
        const ringMid = (outerR + innerR) / 2;

        const fill = isSunny
            ? getComputedStyle(document.documentElement)
                  .getPropertyValue('--primary-color')
                  .trim() || '#f15a26'
            : getComputedStyle(document.documentElement)
                  .getPropertyValue('--secondary-color')
                  .trim() || '#3b9aff';

        svg.append('circle')
            .attr('cx', r)
            .attr('cy', r)
            .attr('r', ringMid)
            .attr('fill', 'none')
            .attr('stroke', fill)
            .attr('stroke-width', ringWidth);
    }

    onMount(() => {
        drawDonut();
    });

    $effect(() => {
        // re-render when relevant inputs change
        ringColor;
        size;
        drawDonut();
    });
</script>

<div
    class="day-cell"
    role="button"
    tabindex="0"
    onmouseenter={(e) => onhover?.(e, dayData)}
    onmousemove={(e) => onmove?.(e, dayData)}
    onmouseleave={() => onleave?.()}
>
    <span class="day-num">{dayNum}</span>

    <div class="ring-wrap" style="width:{size}px;height:{size}px;">
        <svg
            class="donut-svg"
            bind:this={svgEl}
            width={size}
            height={size}
            viewBox="0 0 {size} {size}"
        ></svg>
        <img
            class="cell-shape"
            class:energized={feeling === 'Energized'}
            src={cellShape}
            alt="feeling shape"
        />
        <img class="bpm-bar" src={bpmImg} alt="bpm bar" />
    </div>

   
</div>

<style>
    .day-cell {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 6px;
        padding: 10px;
        border-radius: 6px;
        cursor: pointer;
        transition: transform 0.18s ease;
        outline: none;
    }

    .day-cell:hover,
    .day-cell:focus-visible {
        transform: scale(1.06);
    }

    .day-num {
        font-family: var(--font-leaguespartan);
        font-size: 0.75rem;
        letter-spacing: 0.06em;
        color: var(--black-sub);
        opacity: 0.75;
        /* padding-top: 10px; */
        padding-bottom: 10px;
    }

    .ring-wrap {
        position: relative;
    }

    .ring-wrap > * {
        position: absolute;
        inset: 0;
        width: 100%;
        height: 100%;
        display: block;
        pointer-events: none;
    }

    .donut-svg {
        z-index: 1;
    }

    .cell-shape {
        object-fit: contain;
        z-index: 2;
        opacity: 0.95;
        /* Scale up beyond the ring-wrap bounds, kept centered */
        width: 160%;
        height: 160%;
        inset: auto;
        top: 50%;
        left: 50%;
        --cell-shape-shift-x: 0px;
        transform: translate(calc(-50% + var(--cell-shape-shift-x)), -50%);
    }

    .cell-shape.energized {
        --cell-shape-shift-x: -8px;
    }

    .bpm-bar {
        object-fit: contain;
        z-index: 3;
    }
</style>
