# crypto_wallet_1

import 'package:flutter/material.dart';
import 'package:web3dart/web3dart.dart';
import 'package:http/http.dart';

void main() => runApp(CryptoWalletApp());

class CryptoWalletApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Crypto Wallet',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: CryptoWalletPage(),
    );
  }
}

class CryptoWalletPage extends StatefulWidget {
  @override
  _CryptoWalletPageState createState() => _CryptoWalletPageState();
}

class _CryptoWalletPageState extends State<CryptoWalletPage> {
  final List<Transaction> transactions = [
    Transaction('Bitcoin', 'Received', '+0.5 BTC'),
    Transaction('Ethereum', 'Sent', '-2 ETH'),
    Transaction('Bitcoin', 'Sent', '-1 BTC'),
  ];

  List<Token> tokens = [];

  final String infuraUrl = 'https://mainnet.infura.io/v3/your-infura-project-id';
  final String walletAddress = 'your-wallet-address';

  @override
  void initState() {
    super.initState();
    fetchTokenBalances();
  }

  Future<void> fetchTokenBalances() async {
    final httpClient = Client();
    final ethClient = Web3Client(infuraUrl, httpClient);
    final contractAddress = EthereumAddress.fromHex('token-contract-address');

    final abiCode = '[{"constant":true,"inputs":[{"name":"_owner","type":"address"}],'
        '"name":"balanceOf","outputs":[{"name":"","type":"uint256"}],"payable":false,'
        '"stateMutability":"view","type":"function"}]';

    final contract = DeployedContract(ContractAbi.fromJson(abiCode, 'token-name'), contractAddress);
    final balanceFunction = contract.function('balanceOf');
    final call = ethClient.call(contract: contract, function: balanceFunction, params: [EthereumAddress.fromHex(walletAddress)]);

    final balance = await call;
    final balanceValue = balance.first as BigInt;
    final balanceString = balanceValue.toString();

    setState(() {
      tokens.add(Token('Token Name', 'TOKEN', balanceString));
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Crypto Wallet'),
      ),
      body: Column(
        children: [
          Container(
            padding: EdgeInsets.all(16.0),
            color: Colors.blue,
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  'Your Balance',
                  style: TextStyle(
                    color: Colors.white,
                    fontSize: 24.0,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                SizedBox(height: 8.0),
                Text(
                  '3.5 BTC',
                  style: TextStyle(
                    color: Colors.white,
                    fontSize: 32.0,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
            ),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: tokens.length,
              itemBuilder: (context, index) {
                final token = tokens[index];
                return ListTile(
                  leading: CircleAvatar(
                    child: Text(token.symbol),
                  ),
                  title: Text(token.name),
                  subtitle: Text(token.balance),
                );
              },
            ),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: transactions.length,
              itemBuilder: (context, index) {
                final transaction = transactions[index];
                return ListTile(
                  leading: Icon(Icons.compare_arrows),
                  title: Text(transaction.currency),
                  subtitle: Text(transaction.type),
                  trailing: Text(transaction.amount),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}


class Transaction {
  final String currency;
  final String type;
  final String amount;

  Transaction(this.currency, this.type, this.amount);
}

class Token {
  final String name;
  final String symbol;
  final String balance;

  Token(this.name, this.symbol, this.balance);
}
