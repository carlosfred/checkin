<script>
	import { configureChains, createClient } from "@wagmi/core";
	import { disconnect, watchAccount, signMessage } from "@wagmi/core";
	import { goerli } from "@wagmi/core/chains";
	import { Web3Modal } from "@web3modal/html";
	import { EthereumClient, modalConnectors, walletConnectProvider} from "@web3modal/ethereum";
	import { ethers } from 'ethers';
	import "../app.css";
	
	const config = {
		conta: '',
		assinatura: '',
		mensagem: '',
		verificada: false
	}

	let confirmacaoAssinatura = '';

	const projectId = "1d52caf3465601dbb97709f7f7cf9ee2";

	const chains = [goerli];

	// Wagmi Core Client
	const { provider } = configureChains(chains, [walletConnectProvider({ projectId: projectId })]);
	
	const wagmiClient = createClient({
		autoConnect: false,
		connectors: modalConnectors({
			projectId: projectId,
			version: "1", // or "2"
			appName: "web3Connect",
			chains,
		}),
		provider,
	});

	// Web3Modal and Ethereum Client
	const ethereumClient = new EthereumClient(wagmiClient, chains);
	
	const web3modal = new Web3Modal(
		{ projectId: projectId },
		ethereumClient
	);

	web3modal.setTheme({
		themeMode: "dark",
		themeColor: "blue",
		themeBackground: "gradient",
	});

	const handleConnect = async () => {
		web3modal.openModal();
	}

	const handleDisconnect = async () => {
		await disconnect();
		config.conta = '';
		config.assinatura = '';
		config.mensagem = '';
		config.verificada = '';
		confirmacaoAssinatura = '';
	}

	const handleSignIn = async () => {
		let msg = config.mensagem;
		const signature = await signMessage({
  			message: msg
		})	
		config.assinatura = signature;	
	}

	const handleVerifySignature = async () => {
		const retorno = ethers.utils.verifyMessage(config.mensagem, config.assinatura);
		if (retorno == config.conta) {
			confirmacaoAssinatura = "Assinatura OK";
			config.verificada = true;
		} else {
			confirmacaoAssinatura = "Assinatura Inválida";
			config.verificada = false;
		}
	}

	const unwatch = watchAccount((account) => {
		config.conta = account.address;
	})

</script>

<div class="container mx-auto flex flex-col justify-center items-center">
	<div class="tracking-widest p-4 border-2 border-zinc-100 rounded-lg my-2">{config.conta ?? 'Carteira Não Conectada'}</div>

	{#if !config.conta}
		<button on:click={handleConnect} class="border-2 mt-5 mb-5 py-2 px-4 rounded-lg  border-red-900 bg-red-900 text-white font-semibold">
			Connectar a Carteira V1
		</button>
	{:else}
		<button on:click={handleDisconnect} class="border-2 mt-5 mb-5 py-2 px-4 rounded-lg  border-red-900 bg-red-900 text-white font-semibold">
			Desconnectar a Carteira V1
		</button>
	{/if}
	<div class='flex flex-col justify-center items-center space-y-4 m-4'>
		<h4 class='font-semibold'>mensagem</h4>
		<input type='text' class='border-2 border-gray-700 rounded-md py-2 px-4 w-[400px]' bind:value={config.mensagem}/>
	</div>
	<button on:click={handleSignIn} class="border-2 py-2 px-4 rounded-lg  border-red-900 bg-red-900 text-white font-semibold">
		Assinar Mensagem
	</button>
	<div class="tracking-widest p-4 border-2 border-zinc-100 rounded-lg my-2 max-w-lg">
		{config.assinatura.substring(0,10)}...{config.assinatura.substring(config.assinatura.length-10, config.assinatura.length)}
	</div>
	<button on:click={handleVerifySignature} class="border-2 py-2 px-4 rounded-lg mt-4 border-green-900 bg-green-900 text-white font-semibold">
		Verificar Assinatura
	</button>
	<div class="tracking-widest p-4 border-2 border-zinc-100 rounded-lg my-2 max-w-lg {config.verificada ? 'text-green' : 'text-red'}">
		{confirmacaoAssinatura}
	</div>

</div>
