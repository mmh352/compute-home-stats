<script lang="ts">
    import { onMount, onDestroy, createEventDispatcher } from 'svelte';
    import { Chart, registerables } from 'chart.js';

    Chart.register(...registerables);

    export let labels: string[];
    export let datasets;
    let canvas: HTMLCanvasElement;
    let chart: Chart;
	const dispatch = createEventDispatcher();

    onMount(() => {
        chart = new Chart(canvas.getContext('2d'), {
            type: 'bar',
            data: {
                labels: labels,
                datasets: datasets,
            },
            options: {
                plugins: {
                    legend: {
                        display: false,
                        title: {
                            display: false,
                        }
                    }
                },
                scales: {
                    x: {
                        stacked: true,
                        grid: {
                            display: false,
                        },
                    },
                },
                interaction: {
                    mode: 'index',
                },
                onClick(ev, active) {
                    if (active.length > 0) {
                        dispatch('click', { index: active[0].index });
                    }
                }
            }
        });
    });

    $: {
        if (chart) {
            chart.data.labels = labels;
            chart.data.datasets = datasets;
            chart.update();
        }
    }

    onDestroy(() => {
        chart.destroy();
    });
</script>

<canvas bind:this={canvas} class="h-full w-full max-h-full max-w-full"></canvas>
