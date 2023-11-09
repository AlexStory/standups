<script>
    import { supabase } from "$lib/base";
    import { Button, Card, Checkbox, Modal } from "flowbite-svelte";
    loadData();

    let users = [];
    let tickets = [];
    let selected = [];
    let winnerName = "";
    let winnerModal = false;

    async function loadData() {
        const { data, _error } = await supabase.from("users").select();
        const res = await supabase.from("tickets").select();
        users = data;
        tickets = res.data;
    }

    function toggleUser(name) {
        if (selected.includes(name)) {
            selected = selected.filter((x) => x != name);
        } else {
            selected = [name, ...selected];
        }
    }

    async function createTickets() {
        users.forEach(async (user) => {
            const { error } = await supabase
                .from("tickets")
                .insert({ name: user.name });
        });
    }

    async function clearUsersTickets(user) {
        const { error } = await supabase
            .from("tickets")
            .delete()
            .eq("name", user);
    }

    async function runRaffle() {
        if (tickets.length == 0) {
            await createTickets();
            await loadData();
        }
        const activeTickets = tickets.filter((t) => selected.includes(t.name));
        const winner =
            activeTickets[Math.floor(Math.random() * activeTickets.length)];
        winnerName = winner.name;
        createTickets();
        clearUsersTickets(winner.name);
        winnerModal = true;
    }
</script>

<div class="flex flex-column justify-center min-h-screen">
    <Card class="my-auto">
        <h5>Who's at standup?</h5>
        <div class="flex flex-row flex-wrap">
            {#each users as user (user.id)}
                <div class="mx-2">
                    <Checkbox on:change={toggleUser(user.name)}
                        >{user.name}</Checkbox
                    >
                </div>
            {/each}
        </div>
        <div>
            <Button class="mt-2 float-right" on:click={runRaffle}
                >Choose!</Button
            >
        </div>
    </Card>
</div>

<Modal bind:open={winnerModal} size="lg" green autoclose>
    <div class="text-center">
        <h3>Congrats {winnerName}! You win!</h3>
    </div>
</Modal>
