<script>
  const query = `
query {
	transactions(first: 100, tags: {name: "App-Name", values: ["PublicSquare"]}) {
		edges {
			node { 
				id
				owner { address }
				tags {
					name 
					value
				}
			}
		}
	}	
}
`;

  async function load() {
    const arweave = window.Arweave.init({
      host: "arweave.net",
      port: 443,
      protocol: "https",
    });
    return await arweave.api.post("graphql", { query });
  }

  async function handleSubmit(e) {
    const arweave = window.Arweave.init({});
    // get wallet
    const tx = arweave.createTransaction({ data: e.target.note.value });
    tx.addTag("Content-Type", "text/plain");
    await arweave.transactions.sign(tx);
    await arweave.transactions.post(tx);
  }

  function getPost(tx) {
    const arweave = window.Arweave.init({});
    return arweave.api.get("https://arweave.net/" + tx).then((r) => r.data);
  }
</script>

<form
  class="flex flex-col mb-16 space-y-4"
  on:submit|preventDefault={handleSubmit}
>
  <textarea class="textarea" placeholder="whats happening?" name="note" />
  <button class="btn btn-block"> Post </button>
</form>

{#await load() then result}
  {#each result.data.data.transactions.edges as edge}
    {#await getPost(edge.node.id) then post}
      <div class="border-2 p-4">
        {post}
      </div>
    {/await}
  {/each}
{/await}
