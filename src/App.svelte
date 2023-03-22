<script>
  import { ArweaveWebWallet } from "arweave-wallet-connector";

  const wallet = new ArweaveWebWallet({ name: "Test" });
  wallet.setUrl("arweave.app");

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

  const arweave = window.Arweave.init({
    host: "arweave.net",
    port: 443,
    protocol: "https",
  });
  async function load() {
    return await arweave.api.post("graphql", { query });
  }

  async function handleSubmit(e) {
    if (window.arweaveWallet) {
      await window.arweaveWallet.connect(["DISPATCH"]);
    } else {
      await wallet.connect();
    }
    // get wallet
    const tx = await arweave.createTransaction({ data: e.target.note.value });
    tx.addTag("Content-Type", "text/plain");
    await window.arweaveWallet.dispatch(tx);
    //await arweave.transactions.sign(tx);
    //await arweave.transactions.post(tx);

    console.log({ id: tx.id });
  }

  function getPost(tx) {
    return arweave.api.get(tx).then((r) => r.data);
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
