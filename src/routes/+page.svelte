<script>
    let minutesStored = 1000;
    let minutesDelivered = 1000;
    let averageBitrate = 5000; // in kbps
    let gbStored = calculateGb(minutesStored, averageBitrate);
    let gbDelivered = calculateGb(minutesDelivered, averageBitrate);

    /* Convert minutes and bitrate to GB */
    function calculateGb(minutes, bitrate) {
        const bitsPerSecond = bitrate * 1000; // kbps to bits per second
        const bytesPerMinute = (bitsPerSecond * 60) / 8; // bits to bytes per minute
        const gbPerMinute = bytesPerMinute / 1024 ** 3; // bytes to GB
        return +(gbPerMinute * minutes).toFixed(2); // Rounded to 2 decimal places
    }

    /* Convert GB and bitrate to minutes */
    function calculateMinutes(gb, bitrate) {
        const bitsPerSecond = bitrate * 1000; // kbps to bits per second
        const bytesPerMinute = (bitsPerSecond * 60) / 8; // bits to bytes per minute
        return Math.round((gb * 1024 ** 3) / bytesPerMinute); // Rounded to nearest minute
    }

    // Update GB when minutes or bitrate changes
    function updateGb() {
        gbStored = calculateGb(minutesStored, averageBitrate);
        gbDelivered = calculateGb(minutesDelivered, averageBitrate);
    }

    // Update minutes when GB or bitrate changes
    function updateMinutes() {
        minutesStored = calculateMinutes(gbStored, averageBitrate);
        minutesDelivered = calculateMinutes(gbDelivered, averageBitrate);
    }

    /* Bunny CDN cost calculator */
    const bunnyCDNPrice = () => {
        const storageCost = gbStored * 0.01;
        const deliveryCost = gbDelivered * 0.005;
        const totalCost = storageCost + deliveryCost;

        return {
            storageCost,
            deliveryCost,
            actualCost: totalCost,
            adjustedCost: totalCost < 1 ? 1 : totalCost,
        };
    };

    /* Cloudflare cost calculator */
    const cloudflarePrice = () => {
        const storageBlocks = minutesStored / 1000;
        const storageCost = storageBlocks * 5;
        const deliveryBlocks = minutesDelivered / 1000;
        const deliveryCost = deliveryBlocks * 1;
        const actualCost = storageCost + deliveryCost;

        return {
            storageCost,
            deliveryCost,
            actualCost,
            adjustedCost: Math.ceil(storageCost / 5) * 5 + deliveryCost,
        };
    };
</script>

<h2 style="margin-top: 0.5em;">Bunny Stream vs Cloudflare Stream</h2>

<div style="display: flex; gap: 1em; flex-wrap: wrap;">
    <div>
        <label for="minutesStored">Minutes Stored:</label>
        <input
            type="number"
            id="minutesStored"
            bind:value={minutesStored}
            min="0"
            on:input={updateGb}
        />
    </div>

    <div>
        <label for="minutesDelivered">Minutes Delivered:</label>
        <input
            type="number"
            id="minutesDelivered"
            bind:value={minutesDelivered}
            min="0"
            on:input={updateGb}
        />
    </div>

    <div>
        <label for="averageBitrate">Average Bitrate (kbps):</label>
        <input
            type="number"
            id="averageBitrate"
            bind:value={averageBitrate}
            min="0"
            on:input={() => {
                updateGb();
                updateMinutes();
            }}
        />
    </div>
</div>

<div style="display: flex; gap: 1em; flex-wrap: wrap; margin-top: 1em;">
    <div>
        <label for="gbStored">GB Stored:</label>
        <input
            type="number"
            id="gbStored"
            bind:value={gbStored}
            min="0"
            step="0.01"
            on:input={updateMinutes}
        />
    </div>

    <div>
        <label for="gbDelivered">GB Delivered:</label>
        <input
            type="number"
            id="gbDelivered"
            bind:value={gbDelivered}
            min="0"
            step="0.01"
            on:input={updateMinutes}
        />
    </div>
</div>

<table style="margin-top: 0.5em;">
    <thead>
        <tr>
            <th>Plan</th>
            <th>Storage Cost</th>
            <th>Delivery Cost</th>
            <th>Total Cost</th>
            <th>Rounded Cost</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Bunny Stream</td>
            <td>${bunnyCDNPrice().storageCost.toFixed(2)}</td>
            <td>${bunnyCDNPrice().deliveryCost.toFixed(2)}</td>
            <td>${bunnyCDNPrice().actualCost.toFixed(2)}</td>
            <td>${bunnyCDNPrice().adjustedCost.toFixed(2)}</td>
        </tr>
        <tr>
            <td>Cloudflare Stream</td>
            <td>${cloudflarePrice().storageCost.toFixed(2)}</td>
            <td>${cloudflarePrice().deliveryCost.toFixed(2)}</td>
            <td>${cloudflarePrice().actualCost.toFixed(2)}</td>
            <td>${cloudflarePrice().adjustedCost.toFixed(2)}</td>
        </tr>
    </tbody>
</table>

<div style="margin-top: 1em;">
    <h3>Relative Difference</h3>
    <p>
        {@html cloudflarePrice().actualCost < bunnyCDNPrice().actualCost
            ? `Cloudflare Stream is <b>${(bunnyCDNPrice().actualCost / cloudflarePrice().actualCost).toFixed(2)} times</b> cheaper than Bunny Stream.`
            : `Bunny Stream is <b>${(cloudflarePrice().actualCost / bunnyCDNPrice().actualCost).toFixed(2)} times</b> cheaper than Cloudflare Stream.`}
    </p>
</div>
