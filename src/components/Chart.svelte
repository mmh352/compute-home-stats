<script lang="ts">
    import { onMount, onDestroy } from 'svelte';
    import { Chart, registerables } from 'chart.js';

    Chart.register(...registerables);

    export let labels: string[];
    export let datasets;
    let canvas: HTMLCanvasElement;
    let chart: Chart;

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
                    yAxis: {
                        display: true,
                        min: 0,
                    },
                    xAxis: {
                        grid: {
                            display: false,
                        },
                    },
                },
                interaction: {
                    mode: 'index',
                },
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

<canvas bind:this={canvas} class="max-h-full max-w-full"></canvas>
