<script>
	import { configureChains, createClient } from "@wagmi/core";
	import { disconnect, watchAccount, signMessage, fetchBalance } from "@wagmi/core";
	import { goerli, polygon } from "@wagmi/core/chains";
	import { Web3Modal } from "@web3modal/html";
	import { EthereumClient, modalConnectors, walletConnectProvider} from "@web3modal/ethereum";
	import { ethers } from 'ethers';
	import "../app.css";

	import Logo from "$lib/assets/logo.jpg";

	let signInMessage = "Confirma o Check-in no Bar Bukovisk ?";

	const retorno = [
		{id: 0, msg: 'Check-in Realizado c/ Sucesso !!!'},
		{id: 1, msg: 'Carteira Não possui NFT !!!'},
		{id: 2, msg: 'NFT c/ prazo de validade expirado !!!'},
		{id: 3, msg: 'Assinatura inválida !!!'}
	]

	let idMessagem = -1;
	let mensagemSistema = 'Cliente Não Conectado';

	const config = {
		conta: '',
		assinatura: '',
		mensagem: '',
		saldo: 0,
		verificada: false
	}

	let confirmacaoAssinatura = '';

	const projectId = "1d52caf3465601dbb97709f7f7cf9ee2";

	const chains = [goerli, polygon];

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
		{ projectId: projectId, defaultChain: polygon },
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
		idMessagem = -1;
		config.conta = '';
		config.assinatura = '';
		config.mensagem = '';
		config.verificada = '';
		config.saldo = 0;
		confirmacaoAssinatura = '';
	}

	const handleSignIn = async () => {
		let msg = signInMessage;
		const signature = await signMessage({
  			message: msg
		})	
		config.assinatura = signature;	
	}

	const handleVerifySignature = async () => {
		const retorno = ethers.utils.verifyMessage(config.mensagem, config.assinatura);
		if (retorno == config.conta) {
			idMessagem = Math.floor(Math.random() * 3);
			mensagemSistema = retorno[idMessagem].mensagem;
		} else {
			idMessagem = 3;
			mensagemSistema = retorno[3].mensagem;
		}
	}

	const unwatch = watchAccount((account) => {
		console.log(ethereumClient);
		config.conta = account.address;
		if (config.conta != undefined) {
			handleSignIn();
		}
	})

</script>

<div class="w-screen h-screen bg-black">
	<div class="mx-auto flex flex-col justify-center items-center">
		<div><img src={Logo} alt="Bukovisk Logo"/></div>
		<div class="mt-7">
			<button on:click={handleConnect} class="border-2 mt-5 mb-5 py-2 px-4 rounded-lg  border-red-900 bg-red-900 text-white font-semibold">
				Realizar Check-in
			</button>
		</div>	

		<div class="mt-7">
			<button on:click={handleDisconnect} class="border-2 mt-5 mb-5 py-2 px-4 rounded-lg  border-red-900 bg-red-900 text-white font-semibold">
				Desconnectar a Carteira V1
			</button>			
		</div>

		<div class="mt-7">
			<button on:click={handleVerifySignature} class="border-2 mt-5 mb-5 py-2 px-4 rounded-lg  border-red-900 bg-red-900 text-white font-semibold">
				Verificar a Mensagem
			</button>			
		</div>

		{#if idMessagem > -2}		
			<div class="mt-7 text-center text-white">
				{mensagemSistema}
			</div>
			<div>
				<button on:click={handleDisconnect} class="border-2 mt-5 mb-5 py-2 px-4 rounded-lg  border-red-900 bg-red-900 text-white font-semibold">
					OK
				</button>			
			</div>
		{/if}

	</div>
</div>
