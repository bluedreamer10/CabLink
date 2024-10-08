<!-- MakeRides.svelte -->
<script>
    import { onMount } from "svelte";
    import { navigate } from "svelte-routing";

    let rides = []; // Initialize rides array to store fetched rides
    let startPoint = ""; // State variable for start point filter
    let destination = ""; // State variable for destination filter

    // Fetch rides on component mount
    const fetchRides = async () => {
        try {
            const response = await fetch("/api/ride/all"); // Fetch all rides from backend API
            if (response.ok) {
                const data = await response.json(); // Parse response JSON
                console.log("Fetched rides:", data); // Log fetched data for debugging
                if (data && data.data) {
                    rides = data.data.map((ride) => ({
                        ...ride,
                        _id: ride._id.$oid, // Convert BSON ObjectID to string for serialization
                    })); // Update rides array with fetched data, converting ObjectID
                }
            } else {
                console.error("Failed to fetch rides:", response.statusText); // Log error if fetch fails
            }
        } catch (error) {
            console.error("Error fetching rides:", error); // Log error if fetch throws an exception
        }
    };

    onMount(fetchRides); // Call fetchRides function when component mounts

    // Function to navigate to ride details page with ride ID
    const navigateToRideDetails = async (rideId) => {
        try {
            const response = await fetch(`/api/ride/${rideId}`); // Fetch ride details by ID
            if (response.ok) {
                const rideDetails = await response.json(); // Parse ride details JSON
                console.log("Ride details:", rideDetails); // Log ride details for debugging
                navigate(`/ride/${rideId}`, { state: { ride: rideDetails } }); // Navigate to ride details page with ride details in state
            } else {
                console.error(
                    "Failed to fetch ride details:",
                    response.statusText,
                ); // Log error if fetch fails
            }
        } catch (error) {
            console.error("Error fetching ride details:", error); // Log error if fetch throws an exception
        }
    };

    // Function to format location based on type (assuming location is in GeoJSON format)
    const formatLocation = (location) => {
        switch (location.type) {
            case "Point":
                return `${location.coordinates[0]}, ${location.coordinates[1]}`; // Format coordinates as string
            default:
                return "Unknown Location"; // Return default text for unknown location type
        }
    };

    // Function to handle search rides based on start point and destination (not implemented in this example)
    const searchRides = async () => {
        // Implementation of search functionality can be added here if needed
    };
</script>

<h1>List of Rides</h1>

<div>
    <label for="startPoint">Start Point:</label>
    <select id="startPoint" bind:value={startPoint}>
        <option value="">Select Start Point</option>
        <option value="Campus">Campus</option>
        <option value="Airport">Airport</option>
        <option value="Railway Station">Railway Station</option>
    </select>
</div>

<div>
    <label for="destination">Destination:</label>
    <select id="destination" bind:value={destination}>
        <option value="">Select Destination</option>
        <option value="Campus">Campus</option>
        <option value="Airport">Airport</option>
        <option value="Railway Station">Railway Station</option>
    </select>
</div>

<button on:click={searchRides}>Search Rides</button>

<div class="ride-list">
    {#if rides.length === 0}
        <p>No rides available.</p>
    {:else}
        <ul>
            {#each rides as ride}
                <li
                    class="ride-box"
                    on:click={() => navigateToRideDetails(ride._id)}
                >
                    <div class="ride-details">
                        Name: <span>{ride.name}</span>
                    </div>
                    <div class="ride-details">
                        Start Point: <span
                            >{formatLocation(ride.locationStart)}</span
                        >
                    </div>
                    <div class="ride-details">
                        Destination: <span
                            >{formatLocation(ride.locationEnd)}</span
                        >
                    </div>
                    <div class="ride-details">
                        Date & Time: <span
                            >{new Date(ride.date).toLocaleString()}</span
                        >
                    </div>
                    <div class="ride-details">
                        Host: <span
                            >{ride.host ? ride.host.name : "Unknown Host"}</span
                        >
                    </div>
                </li>
            {/each}
        </ul>
    {/if}
</div>

<style>
    /* CSS styles for ListRides component */
    .ride-list {
        max-height: 400px; /* Limit height of ride list to provide scrollable view */
        overflow-y: auto; /* Enable vertical scrolling */
        padding: 10px;
        border: 1px solid #ccc;
        border-radius: 5px;
    }

    .ride-box {
        background-color: #ffffff;
        border: 1px solid #e0e0e0;
        border-radius: 8px;
        padding: 12px;
        margin-bottom: 10px;
        cursor: pointer;
        transition: background-color 0.3s ease;
    }

    .ride-box:hover {
        background-color: #f5f5f5;
    }

    .ride-details {
        font-weight: bold;
        margin-bottom: 8px;
    }

    .ride-details span {
        font-weight: normal;
    }
</style>
