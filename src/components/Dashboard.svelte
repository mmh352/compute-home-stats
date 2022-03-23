<script lang="ts">
    import { derived, get, writable } from 'svelte/store';
    import Chart from './Chart.svelte';

    const imageNameMap = {
        'mmh352/tt284-block1:21j.1': 'TT284 21J B1',
        'mmh352/tt284-block1:21j.2': 'TT284 21J B1',
        'mmh352/tt284-block1:21j.3': 'TT284 21J B1',
        'mmh352/tt284-block1:21j.4': 'TT284 21J B1',
        'mmh352/tt284-block2:21j.3-b0': 'TT284 21J B2',
        'mmh352/tt284-block2:21j.3': 'TT284 21J B2',
        'mmh352/tt284-block2:21j.4': 'TT284 21J B2',
        'mmh352/tt284-block3:21j.4': 'TT284 21J B3',
        'mmh352/tm129-robotics:21j.0-b2': 'TM129 Robotics 21J',
        'mmh352/tm129-robotics:21j.0-b3': 'TM129 Robotics 21J',
        'mmh352/tm129-robotics:21j.0-b4': 'TM129 Robotics 21J',
        'mmh352/tm129-robotics:21j.0-b5': 'TM129 Robotics 21J',
        'mmh352/tm129-robotics:21j.0-b6': 'TM129 Robotics 21J',
        'mmh352/tm129-robotics:21j.0': 'TM129 Robotics 21J',
        'mmh352/s397-21j:latest': 'S397 21J',
        'mmh352/s397-21j:1': 'S397 21J',
        'mmh352/s397-21j:2': 'S397 21J',
    }
    const colours = {
        'TT284 21J B1': '#0e56a7',
        'TT284 21J B2': '#326fb4',
        'TT284 21J B3': '#5689c1',
        'TM129 Robotics 21J': '#068293',
        'S397 21J': '#e21481',
    }
    const monthLabels = {
        1: 'January',
        2: 'Feburary',
        3: 'March',
        4: 'April',
        5: 'May',
        6: 'June',
        7: 'July',
        8: 'August',
        9: 'September',
        10: 'October',
        11: 'November',
        12: 'December'
    }
    const presentations = [
        {
            label: '21J',
            months: [
                [2021, 9],
                [2021, 10],
                [2021, 11],
                [2021, 12],
                [2022, 1],
                [2022, 2],
                [2022, 3],
                [2022, 4],
                [2022, 5],
                [2022, 6]
            ]
        }
    ]

    const data = writable([]);
    const scale = writable('everything');
    const scaleFilter = writable('');
    const metric = writable('session-counts');

    const labels = derived([scale, scaleFilter], ([scale, scaleFilter]) => {
        if (scale === 'everything') {
            return presentations.map((presentation) => {
                return presentation.label;
            });
        } else if (scale === 'presentation') {
            for (let presentation of presentations) {
                if (presentation.label === scaleFilter) {
                    return presentation.months.map((month) => {
                        return monthLabels[month[1]] + ' ' + month[0];
                    })
                }
            }
            return [];
        } else if (scale === 'year') {
            return ['January', 'Feburary', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'];
        } else if (scale === 'month') {
            return [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31];
        } else if (scale === 'day') {
            return ['00', '01', '02', '03', '04', '05', '06', '07', '08', '09', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23']
        } else {
            return [];
        }
    });

    const scaleFilters = derived([data, scale], ([data, scale]) => {
        if (scale === 'presentation') {
            const filters = presentations.map((presentation) => {
                return {value: presentation.label, label: presentation.label};
            });
            if (filters.length > 0) {
                scaleFilter.set(filters[0].value);
            }
            return filters
        } else if (scale === 'year') {
            const filters = data.reduce((acc, cur) => {
                if (acc.map((val) => { return val.value }).indexOf(cur.year) < 0) {
                    acc.push({value: cur.year, label: cur.year.toString()});
                }
                return acc;
            }, []);
            filters.sort((a, b) => {
                return b.value - a.value;
            });
            if (filters.length > 0) {
                scaleFilter.set(filters[0].value);
            }
            return filters;
        } else if (scale === 'month') {
            const filters = data.reduce((acc, cur) => {
                if (acc.map((val) => { return val.value }).indexOf(cur.year + '.' + cur.month) < 0) {
                    acc.push({value: cur.year + '.' + cur.month, label: monthLabels[cur.month] + ' ' + cur.year});
                }
                return acc;
            }, []);
            if (filters.length > 0) {
                scaleFilter.set(filters[filters.length - 1].value);
            }
            return filters;
        } else if (scale === 'day') {
            const filters = data.reduce((acc, cur) => {
                if (acc.map((val) => { return val.value }).indexOf(cur.year + '.' + cur.month + '.' + cur.day) < 0) {
                    acc.push({value: cur.year + '.' + cur.month + '.' + cur.day, label: cur.day + ' ' + monthLabels[cur.month] + ' ' + cur.year});
                }
                return acc;
            }, []);
            if (filters.length > 0) {
                scaleFilter.set(filters[filters.length - 1].value);
            }
            return filters;
        } else {
            return [];
        }
    });

    function createDataPoint(datapoints, data, metric: string, group: string, constraint) {
        if (metric === 'session-counts') {
            datapoints.push(data.reduce((acc, cur) => {
                if (constraint(cur) && imageNameMap[cur.image] === group && cur.continued === false) {
                    acc = acc + 1;
                }
                return acc;
            }, 0));
        } else if (metric === 'unique-users') {
            datapoints.push(data.reduce((acc, cur) => {
                if (constraint(cur) && imageNameMap[cur.image] === group && acc.indexOf(cur.user) < 0) {
                    acc.push(cur.user);
                }
                return acc;
            }, []).length);
        } else if (metric === 'median-session-lengths') {
            const tmp = {}
            const lengths = data.reduce((acc, cur) => {
                if (constraint(cur) && imageNameMap[cur.image] === group) {
                    if (!cur.continued) {
                        if (tmp[cur.user]) {
                            acc.push(tmp[cur.user] + cur.duration);
                            delete tmp[cur.user];
                        } else {
                            acc.push(cur.duration);
                        }
                    } else {
                        if (tmp[cur.user]) {
                            tmp[cur.user] = tmp[cur.user] + cur.duration;
                        } else {
                            tmp[cur.user] = cur.duration;
                        }
                    }
                }
                return acc;
            }, []);
            if (Object.values(tmp).length > 0) {
                for (const value in Object.values(tmp)) {
                    lengths.push(value);
                }
            }
            if (lengths.length > 0) {
                lengths.sort((a: number, b: number) => { return a - b; });
                if (lengths.length % 2 === 0) {
                    datapoints.push(Math.floor((lengths[Math.floor(lengths.length / 2)] + lengths[Math.ceil(lengths.length / 2)]) / 120));
                } else {
                    datapoints.push(Math.floor(lengths[Math.floor(lengths.length / 2)] / 60));
                }
            } else {
                datapoints.push(0);
            }
        } else {
            datapoints.push(0);
        }
    }

    const datasets = derived([data, scale, scaleFilter, metric], ([data, scale, scaleFilter, metric]) => {
        if (scale === 'everything') {
            const groups = data.map((entry) => {
                if (!imageNameMap[entry.image]) {
                    console.error('Missing image ' + entry.image);
                }
                return imageNameMap[entry.image];
            }).reduce((acc, cur) => { if (acc.indexOf(cur) < 0) { acc.push(cur); } return acc; }, []);
            groups.sort();
            const datasets = groups.map((group: string) => {
                const datapoints = [];
                for (let presentation of presentations) {
                    createDataPoint(datapoints, data, metric, group, (cur) => {
                        const matches = presentation.months.filter((presentationMonth) => { return presentationMonth[0] === cur.year && presentationMonth[1] === cur.month});
                        return matches.length > 0;
                    });
                }
                return {
                    label: group,
                    backgroundColor: colours[group],
                    data: datapoints,
                }
            });
            return datasets;
        } else if (scale === 'presentation') {
            const groups = data.map((entry) => {
                if (!imageNameMap[entry.image]) {
                    console.error('Missing image ' + entry.image);
                }
                return imageNameMap[entry.image];
            }).reduce((acc, cur) => { if (acc.indexOf(cur) < 0) { acc.push(cur); } return acc; }, []);
            groups.sort();
            const presentation = presentations.filter((presentation) => { return presentation.label === scaleFilter})[0];
            const datasets = groups.map((group: string) => {
                const datapoints = [];
                for (let presentationMonth of presentation.months) {
                    createDataPoint(datapoints, data, metric, group, (cur) => {
                        return presentationMonth[0] === cur.year && presentationMonth[1] === cur.month;
                    });
                }
                return {
                    label: group,
                    backgroundColor: colours[group],
                    data: datapoints,
                }
            });
            return datasets;
        } else if (scale === 'year') {
            const groups = data.map((entry) => { return imageNameMap[entry.image]; }).reduce((acc, cur) => { if (acc.indexOf(cur) < 0) { acc.push(cur); } return acc; }, []);
            groups.sort();
            const datasets = groups.map((group: string) => {
                const datapoints = [];
                for (let month = 1; month <= 12; month++) {
                    createDataPoint(datapoints, data, metric, group, (cur) => {
                        return scaleFilter === cur.year && cur.month === month;
                    });
                }
                return {
                    label: group,
                    backgroundColor: colours[group],
                    data: datapoints,
                }
            });
            return datasets;
        } else if (scale === 'month') {
            const groups = data.map((entry) => { return imageNameMap[entry.image] }).reduce((acc, cur) => { if (acc.indexOf(cur) < 0) { acc.push(cur); } return acc; }, []);
            groups.sort();
            const filterYear = Number.parseInt(scaleFilter.substring(0, scaleFilter.indexOf('.')));
            const filterMonth = Number.parseInt(scaleFilter.substring(scaleFilter.indexOf('.') + 1));
            const datasets = groups.map((group: string) => {
                const datapoints = [];
                for (let day = 1; day <= 31; day++) {
                    createDataPoint(datapoints, data, metric, group, (cur) => {
                        return filterYear === cur.year && filterMonth == cur.month && cur.day === day;
                    });
                }
                return {
                    label: group,
                    backgroundColor: colours[group],
                    data: datapoints,
                }
            });
            return datasets;
        } else if (scale === 'day') {
            const groups = data.map((entry) => { return imageNameMap[entry.image] }).reduce((acc, cur) => { if (acc.indexOf(cur) < 0) { acc.push(cur); } return acc; }, []);
            groups.sort();
            const filterYear = Number.parseInt(scaleFilter.substring(0, scaleFilter.indexOf('.')));
            const tmp = scaleFilter.substring(scaleFilter.indexOf('.') + 1);
            const filterMonth = Number.parseInt(tmp.substring(0, tmp.indexOf('.')));
            const filterDay = Number.parseInt(tmp.substring(tmp.indexOf('.') + 1));
            const datasets = groups.map((group: string) => {
                const datapoints = [];
                for (let hour = 0; hour <= 23; hour++) {
                    createDataPoint(datapoints, data, metric, group, (cur) => {
                        return filterYear === cur.year && filterMonth == cur.month && filterDay === cur.day && cur.hour === hour;
                    });
                }
                return {
                    label: group,
                    backgroundColor: colours[group],
                    data: datapoints,
                }
            });
            return datasets;
        } else {
            return [];
        }
    });

    function scaleFilterPrev() {
        const scaleFilterList = get(scaleFilters);
        const idx = scaleFilterList.map((val) => { return val.value; }).indexOf(get(scaleFilter));
        if (idx > 0) {
            scaleFilter.set(scaleFilterList[idx - 1].value);
        }
    }

    function scaleFilterNext() {
        const scaleFilterList = get(scaleFilters);
        const idx = scaleFilterList.map((val) => { return val.value; }).indexOf(get(scaleFilter));
        if (idx < scaleFilterList.length - 1) {
            scaleFilter.set(scaleFilterList[idx + 1].value);
        }
    }

    async function load() {
        const response = await fetch('time-entries.json');
        data.set(await response.json());
    }

    function chartClick(ev) {
        if (get(scale) === 'everything') {
            scale.set('presentation');
            scaleFilter.set(presentations[ev.detail.index].label);
        } else if (get(scale) === 'presentation') {
            const presentation = presentations.filter((presentation) => { return presentation.label === get(scaleFilter)})[0];
            scale.set('month');
            scaleFilter.set(presentation.months[ev.detail.index][0] + '.' + presentation.months[ev.detail.index][1]);
        } else if (get(scale) === 'year') {
            const newFilter = get(scaleFilter) + '.' + (ev.detail.index + 1);
            scale.set('month');
            scaleFilter.set(newFilter);
        } else if (get(scale) === 'month') {
            const newFilter = get(scaleFilter) + '.' + (ev.detail.index + 1);
            scale.set('day');
            scaleFilter.set(newFilter);
        }
    }

    load();
</script>

<div class="w-full h-full flex flex-col overflow-hidden">
    <nav class="flex-0 mb-2">
        <ul class="flex flex-row items-center">
            <li class="mr-6">
                <select bind:value={$scale} class="px-2 py-1">
                    <option value="everything">Everything</option>
                    <option value="presentation">Presentation</option>
                    <option value="year">Year</option>
                    <option value="month">Month</option>
                    <option value="day">Day</option>
                </select>
            </li>
            {#if $scaleFilters.length > 0}
                <li>
                    <button on:click={() => { if ($scale === 'year') { scaleFilterNext(); } else { scaleFilterPrev(); }}} class="block px-3 py-1">
                        <svg viewBox="0 0 24 24" class="block w-4 h-4">
                            <path fill="currentColor" d="M15.41,16.58L10.83,12L15.41,7.41L14,6L8,12L14,18L15.41,16.58Z" />
                        </svg>
                    </button>
                </li>
                <li>
                    <select bind:value={$scaleFilter} class="px-2 py-1">
                        {#each $scaleFilters as entry}
                            <option value={entry.value}>{entry.label}</option>
                        {/each}
                    </select>
                </li>
                <li>
                    <button on:click={() => { if ($scale === 'year') { scaleFilterPrev(); } else { scaleFilterNext(); }}} class="block px-3 py-1">
                        <svg viewBox="0 0 24 24" class="block w-4 h-4">
                            <path fill="currentColor" d="M8.59,16.58L13.17,12L8.59,7.41L10,6L16,12L10,18L8.59,16.58Z" />
                        </svg>
                    </button>
                </li>
            {/if}
            <li class="ml-6">
                <select bind:value={$metric} class="px-2 py-1">
                    <option value="session-counts">Session Counts</option>
                    <option value="unique-users">Unique Users</option>
                    <option value="median-session-lengths">Median Session Lengths (minutes)</option>
                </select>
            </li>
        </ul>
    </nav>
    <div class="flex-auto overflow-hidden">
        <Chart labels={$labels} datasets={$datasets} on:click={chartClick} />
    </div>
</div>
