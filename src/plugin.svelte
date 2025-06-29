<div class="plugin__mobile-header">
    {title}
</div>
<section class="plugin__content">
    <div
        class="plugin__title plugin__title--chevron-back"
        on:click={() => bcast.emit('rqstOpen', 'menu')}
    >
        {title}
    </div>

    <p class="mt-30 mb-30">
        <img src="https://www.windy.com/img/windy-plugins/borat-great-success-ed.png" alt="Borat" />
    </p>

    <p class="size-l">Congratulations, you have just launched your first Windy plugin!</p>

    <p>
        This is example of standard <code>desktopUI: 'rhpane'</code> plugin layout. Default width of
        the plugin is 400px, but it can be changed by setting <code>desktopWidth</code> property in
        <code>pluginConfig.ts</code> file.
    </p>

    <p>Please allow GPS location in your browser to see your location on the map.</p>
    <div class="centered m-15">
        <button
            class="button button--variant-orange"
            class:button--loading={loader}
            on:click={getMyLoc}
        >
            Show my location
        </button>
    </div>

    <p>Perform API call.</p>
    <div class="centered m-15">
        <button
            class="button button--variant-orange"
            class:button--loading={loader}
            on:click={getNasaApi}
        >
            NASA API Call
        </button>
    </div>

    <hr />

    {#each [1, 2, 3, 4, 5, 6, 7, 8, 9] as _line}
        <p>
            The major advantage of rhpane and mobile fullscreen layout is ability to display a lot
            of information in a single view.
        </p>
        <p>
            Overflowed content is scrollable and it works like a charm especially on mobile devices.
        </p>
    {/each}
</section>

<script lang="ts">
    import bcast from '@windy/broadcast';
    import { map, markers } from '@windy/map';
    import { getGPSlocation } from '@windy/geolocation';
    import { onDestroy } from 'svelte';

    import config from './pluginConfig';

    const { title } = config;

    let marker: L.Marker | null = null;
    let loader = false;

    const getMyLoc = async () => {
        loader = true;
        const loc = await getGPSlocation();
        loader = false;
        if (loc) {
            const { lat, lon: lng } = loc;
            map.setView({ lat, lng }, 8);
            marker = new L.Marker({ lat, lng }, { icon: markers.myLocationIcon }).addTo(map);
        }
    };

    enum ConfidenceRating {
        High,
        Low,
    }

    enum DayNight {
        Day,
        Night,
    }

    // to represent a potential fire
    type PotentialFireLocation = {
        latitide: number;
        longitude: number;
        path: number;
        row: number;
        scan: number;
        track: number;
        acquired_date: Date;
        acquired_time: number;
        satellite: string;
        confidence: ConfidenceRating;
        day_night: DayNight;
    };

    type PotentialFireLocation_VIIRS_S_NPP = {
        latitide: number;
        longitude: number;
        bright_ti4: number;
        scan: number;
        track: number;
        acquired_date: Date;
        acquired_time: number;
        satellite: string;
        instrument: string;
        confidence: ConfidenceRating;
        version: number; // version number
        bright_ti5: number;
        frp: number;
        day_night: DayNight;
    };
    const potential_fire_location_num_values = 14; // probably a better way to do this

    const getNasaApi = async () => {
        let uri: string =
            '';

        const response = await fetch(uri);
        let row_delimiter = '\n';
        let delimiter = ',';
        let rows_array: string[] = (await response.text()).split(row_delimiter);
        let rows: string = rows_array[0]; // why does typescript put the array inside another array??
        console.log(rows_array);
        console.log('rows = ' + rows_array);
        console.log('rows.length = ' + rows.length);

        let potential_fires = [];
        let num_csv_header_values = 0;

        console.log('rows_array.length= ' + rows_array.length);
        // while we still have another delimiter
        for (let i = 0; i < rows_array.length; i++) {
            // console.log('DEBUG: row = ' + rows_array[i]);
            let columns = rows_array[i].split(delimiter);

            // console.log('columns.length= ' + columns.length);
            // console.log('num_csv_header_values=' + num_csv_header_values);
            // console.log(
            // 'potential_fire_location_num_values= ' + potential_fire_location_num_values,
            // );

            // first time around is the header, ignore it
            if (i == 0) {
                num_csv_header_values = columns.length;
                continue;
            }

            // if the number of header values is invalid
            if (
                columns.length != num_csv_header_values ||
                columns.length != potential_fire_location_num_values
            ) {
                console.log(
                    'WARNING: skipped row ' + i + ' due to missing/extra values than expected',
                );
                continue;
            }

            // build the PotentialFireLocation
            // FIXME: this is not actually using the PotentialFire type
            // let potential_fire = {
            //     latitide: columns[0],
            //     longitude: columns[1],
            //     path: columns[2],
            //     row: columns[3],
            //     scan: columns[4],
            //     track: columns[5],
            //     acquired_date: columns[6],
            //     acquired_time: columns[7],
            //     satellite: columns[8],
            //     confidence: columns[9],
            //     day_night: columns[10],
            // };

            let potential_fire = {
                latitide: columns[0],
                longitude: columns[1],
                bright_ti4: columns[2],
                scan: columns[3],
                track: columns[4],
                acquired_date: columns[5],
                acquired_time: columns[6],
                satellite: columns[7],
                instrument: columns[8],
                confidence: columns[9],
                version: columns[10],
                bright_ti5: columns[11],
                frp: columns[12],
                day_night: columns[13],
            };

            potential_fires.push(potential_fire);
            // console.log(potential_fires);
        }

        // FIXME: this is super laggy, we probably want to only display markers on screen
        for (let potential_fire of potential_fires) {
            let lat = +potential_fire.latitide;
            let lng = +potential_fire.longitude;
            // map.setView({ lat, lng }, 8);
            marker = new L.Marker({ lat, lng }, { icon: markers. });

            // add a tooltip with just the latitude and longitude for now
            marker.bindTooltip(potential_fire.latitide + potential_fire.longitude);
            marker.addTo(map);
        }
    };

    onDestroy(() => {
        // Your plugin will be destroyed
        // Make sure you cleanup after yourself
        if (marker) {
            marker.remove();
            marker = null;
        }
    });
</script>

<style lang="less">
    p {
        line-height: 1.8;
    }
    code {
        color: lightgray;
    }
    img {
        display: block;
        width: 70%;
        margin: 0 auto;
    }
</style>
