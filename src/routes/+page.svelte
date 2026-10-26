<script>
    import { onMount } from 'svelte';
    import Quote from '#lib/components/Quote.svelte';

    let myQuotes = [
        { id: 'a', text: "Don't be Captain Obvious!", author: 'Paul Opitz' },
        { id: 'b', text: 'You shall not pass!', author: 'Gandalf' },
        { id: 'c', text: 'Not all those who wander are lost.', author: 'Bilbo Baggins' },
        { id: 'd', text: 'Hi, my name is, what? My name is, who? My name is, chka-chka, Slim Shady.', author: 'Eminem' },
        { id: 'e', text: 'I am the one who knocks!', author: 'Walter White' },
        { id: 'f', text: 'I am Groot.', author: 'Groot' },
        { id: 'g', text: 'To infinity and beyond!', author: 'Buzz Lightyear' },
        { id: 'h', text: 'I am Iron Man.', author: 'Tony Stark' },
        { id: 'i', text: 'With great power comes great responsibility.', author: 'Uncle Ben' },
        { id: 'j', text: 'I am inevitable.', author: 'Thanos' }
    ];

    const easterEggQuote = {
        id: 'easter-egg',
        text: 'You found the secret location! Welcome to the centre of the hall.',
        author: 'The Easter Egg'
    };

    let isAtLocation = $state(false);
    let distance = $state(null);
    let locationError = $state('');

    const schoolLocation = {
        latitude: 42.05910523134263,
        longitude: 19.517268301553923
    };

    const radius = 15;

    function calculateDistance(lat1, lon1, lat2, lon2) {
        const earthRadius = 6371000;
        const toRadians = (degrees) => degrees * Math.PI / 180;

        const dLat = toRadians(lat2 - lat1);
        const dLon = toRadians(lon2 - lon1);

        const a =
            Math.sin(dLat / 2) ** 2 +
            Math.cos(toRadians(lat1)) *
            Math.cos(toRadians(lat2)) *
            Math.sin(dLon / 2) ** 2;

        return earthRadius * 2 * Math.atan2(
            Math.sqrt(a),
            Math.sqrt(1 - a)
        );
    }

    onMount(() => {
        if (!navigator.geolocation) {
            locationError = 'Your browser does not support GPS location.';
            return;
        }

        const watchId = navigator.geolocation.watchPosition(
            (position) => {
                const { latitude, longitude, accuracy } = position.coords;

                distance = calculateDistance(
                    latitude,
                    longitude,
                    schoolLocation.latitude,
                    schoolLocation.longitude
                );

                isAtLocation = distance <= radius + accuracy * 0;

                locationError = '';
            },
            (error) => {
                if (error.code === 1) {
                    locationError = 'Please allow location access to find the Easter egg.';
                } else {
                    locationError = 'Unable to determine your location. Please try again.';
                }
            },
            {
                enableHighAccuracy: true,
                maximumAge: 0,
                timeout: 15000
            }
        );

        return () => navigator.geolocation.clearWatch(watchId);
    });

    function dragstart(event) {
        console.log('start');
        event.dataTransfer.setData('text/plain', event.currentTarget.id);
    }

    function dragover(event) {
        event.preventDefault();
    }

    function drop(event) {
        event.preventDefault();

        const id = event.dataTransfer.getData('text/plain');
        const draggedElement = document.getElementById(id);
        const dropZone = event.currentTarget;

        if (draggedElement && dropZone && !draggedElement.contains(dropZone)) {
            dropZone.appendChild(draggedElement);
        }

        dropZone.classList.remove('bg-green-100');
    }
</script>

<svelte:head>
    <title>Data Exchange - Easter Egg</title>
</svelte:head>

<h1 class="m-6 text-center text-4xl">Data Exchange</h1>

{#if isAtLocation}
    <div class="m-4 rounded-xl border-2 border-yellow-400 bg-yellow-100 p-6 text-center shadow-lg">
        <h2 class="mb-3 text-2xl font-bold text-yellow-900">
            🎉 Easter Egg Unlocked!
        </h2>

        <Quote author={easterEggQuote.author}>
            {easterEggQuote.text}
        </Quote>
    </div>
{:else}
    <div class="m-4 rounded-lg bg-gray-100 p-4 text-center">
        <h2 class="text-lg font-semibold">🔍 Find the Easter Egg</h2>

        {#if distance !== null}
            <p class="mt-2">
                You are approximately {Math.round(distance)} metres from the secret location.
            </p>

            <p class="mt-1 text-sm text-gray-600">
                Get within 15 metres of the centre of the school hall.
            </p>
        {:else}
            <p class="mt-2 text-sm text-gray-600">
                Waiting for your location...
            </p>
        {/if}

        {#if locationError}
            <p class="mt-2 text-sm text-red-600">{locationError}</p>
        {/if}
    </div>
{/if}

<main class="flex">
    <div
        class="flex w-1/2 flex-col gap-4 bg-red-100 p-4"
        ondragover={dragover}
        ondrop={drop}
    >
        {#each myQuotes as quote (quote.id)}
            <div
                draggable="true"
                ondragstart={dragstart}
                id={quote.id}
            >
                <Quote author={quote.author}>
                    {quote.text}
                </Quote>
            </div>
        {/each}
    </div>

    <div
        class="flex w-1/2 flex-col gap-4 bg-green-100 p-4"
        ondragover={dragover}
        ondrop={drop}
    ></div>
</main>