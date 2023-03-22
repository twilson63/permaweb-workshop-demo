<script>
  import {
    DeployPlugin,
    InjectedArweaveSigner,
    // @ts-ignore
  } from "https://unpkg.com/warp-contracts-plugin-deploy@1.0.1/bundles/web.bundle.min.js";
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
    warp = window.warp.WarpFactory.forMainnet().use(new DeployPlugin());
    if (window.arweaveWallet) {
      await window.arweaveWallet.connect(["DISPATCH"]);
    } else {
      await wallet.connect();
    }
    // get wallet
    const address = await window.arweaveWallet.getActiveAddress();
    const tx = await arweave.createTransaction({ data: e.target.note.value });
    tx.addTag("Content-Type", "text/plain");
    tx.addTag("App-Name", "PublicSquare");
    tx.addTag("Version", "1.0.1");
    tx.addTag("Type", "post");
    tx.addTag("App-Name", "SmartWeaveContract");
    tx.addTag("App-Version", "0.3.0");
    tx.addTag("Contract-Src", "x0ojRwrcHBmZP20Y4SY0mgusMRx-IYTjg5W8c3UFoNs");
    tx.addTag(
      "Init-State",
      JSON.stringify({
        pairs: [],
        balances: {
          [address]: 1 * 1e12,
        },
        name: "Atomic Token",
        ticker: "ATOMIC",
        settings: [["isTradeable", true]],
      })
    );

    const result = await window.arweaveWallet.dispatch(tx);
    await warp.register(result.id, "node2");
    //await arweave.transactions.sign(tx);
    //await arweave.transactions.post(tx);

    console.log({ id: result.id });
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
