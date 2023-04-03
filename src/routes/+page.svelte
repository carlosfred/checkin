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

	const retornoStatus = [
		{id: 0, msg: 'Check-in Realizado c/ Sucesso !!!'},
		{id: 1, msg: 'Carteira Não possui NFT !!!'},
		{id: 2, msg: 'NFT c/ prazo de validade expirado !!!'},
		{id: 3, msg: 'Assinatura inválida !!!'}
	]

	let idMensagem = -1;
	let mensagemSistema = 'Cliente Não Conectado';

	const config = {
		conta: '',
		assinatura: '',
		mensagem: '',
		saldo: 0,
		verificada: false
	}

	const projectId = "1d52caf3465601dbb97709f7f7cf9ee2";

	const chains = [polygon, goerli];

	// Wagmi Core Client
	const { provider } = configureChains(chains, [walletConnectProvider({ projectId })]);

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
		{ projectId: projectId, defaultChain: polygon, explorerAllowList: [] },
		ethereumClient
	);

	web3modal.setTheme({
		themeMode: "dark",
		themeColor: "blue",
		themeBackground: "gradient",
	});

	const handleConnect = async () => {
		handleDisconnect();
		web3modal.openModal();
	}

	const handleDisconnect = async () => {
		await disconnect();
		idMensagem = -1;
		config.conta = '';
		config.assinatura = '';
		config.mensagem = '';
		config.verificada = '';
		config.saldo = 0;
	}

	const handleSignIn = async () => {
		let agora = new Date().getTime();
		let msg = signInMessage + "(" + agora + ")";
		await signMessage({
  			message: msg
		}).then ((signature) => {
			config.assinatura = signature;	
			handleVerifySignature(msg);
		})
		.catch(error => {
			idMensagem = 4;
			mensagemSistema = "Ocorreu um erro na validação da assinatura, tente novamente !!!";
    	})	
	}

	const handleVerifySignature = async (msg) => {
		web3modal.closeModal();
		const ret = ethers.utils.verifyMessage(msg, config.assinatura);
		if (ret == config.conta) {
			idMensagem = Math.floor(Math.random() * 3);
			mensagemSistema = retornoStatus[idMensagem].msg;
		} else {
			idMensagem = 3;
			mensagemSistema = retornoStatus[3].msg;
		}
	}

	const unwatch = watchAccount((account) => {
		config.conta = account.address;
		if (config.conta != undefined) {
			idMensagem = 0;
			mensagemSistema = "Aguardando validação do Cliente !!!";
			handleSignIn();
		}
	})

</script>

<div class="w-screen h-screen bg-black">
	<div class="mx-auto flex flex-col justify-center items-center">
		<div><img src={Logo} alt="Bukovisk Logo"/></div>
		{#if idMensagem == -1}
			<div class="mt-7">
				<button on:click={handleConnect} class="border-2 mt-5 mb-5 py-2 px-4 rounded-lg  border-red-900 bg-red-900 text-white font-semibold">
					Realizar Check-in
				</button>
			</div>	
		{/if}

		{#if idMensagem > -1}		
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
