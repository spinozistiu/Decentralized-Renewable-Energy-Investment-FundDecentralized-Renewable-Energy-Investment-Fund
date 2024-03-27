# Decentralized-Renewable-Energy-Investment-FundDecentralized-Renewable-Energy-Investment-Fund
Build a WEB3-based investment fund that allows individuals to invest in and support renewable energy projects worldwide.
from web3 import Web3
from solcx import compile_standard, install_solc
import json

# Install Solidity compiler
install_solc('0.8.0')

# Compile Solidity contract
with open('InvestmentFund.sol', 'r') as file:
    investment_fund_source = file.read()

compiled_sol = compile_standard({
    "language": "Solidity",
    "sources": {"InvestmentFund.sol": {"content": investment_fund_source}},
    "settings": {
        "outputSelection": {
            "*": {
                "*": ["abi", "metadata", "evm.bytecode", "evm.sourceMap"]
            }
        }
    }
},
solc_version='0.8.0')

# Extract ABI and bytecode
abi = compiled_sol['contracts']['InvestmentFund.sol']['InvestmentFund']['abi']
bytecode = compiled_sol['contracts']['InvestmentFund.sol']['InvestmentFund']['evm']['bytecode']['object']

# Connect to Ganache (or any Ethereum testnet)
w3 = Web3(Web3.HTTPProvider('http://127.0.0.1:8545'))
w3.eth.defaultAccount = w3.eth.accounts[0]  # Default to the first ganache account for demo

# Deploy the contract
InvestmentFund = w3.eth.contract(abi=abi, bytecode=bytecode)
tx_hash = InvestmentFund.constructor().transact()
tx_receipt = w3.eth.wait_for_transaction_receipt(tx_hash)

# Create contract instance
fund = w3.eth.contract(address=tx_receipt.contractAddress, abi=abi)

# Demonstration of interacting with the contract
# Investing
tx_hash = fund.functions.invest().transact({'value': w3.toWei(1, 'ether'), 'from': w3.eth.accounts[1]})
w3.eth.wait_for_transaction_receipt(tx_hash)

# Withdrawing
tx_hash = fund.functions.withdraw(w3.toWei(0.5, 'ether')).transact({'from': w3.eth.accounts[1]})
w3.eth.wait_for_transaction_receipt(tx_hash)

print(f"Balance of account 1: {w3.eth.get_balance(w3.eth.accounts[1])} Wei")

# Additional functions for interacting with the contract can be added as needed
