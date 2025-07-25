# whatsabi-py

A python implementation of [WhatsABI](https://github.com/shazow/whatsabi).

# install

```
pip install git+https://github.com/viper7882/whatsabi-py.git
```

# Upgrade to latest Web3
1. Open your pyproject.toml file.
2. Find the section where web3 is listed under [tool.poetry.dependencies].
3. Change the version of web3 to "*" to get the latest version or specify the latest version number if you know it.
4. Save the pyproject.toml file.
5. Run poetry update web3 in your terminal to update the lock file and install the new version.
```shell
poetry update web3
```

# cli usage
Pull the code:
```sh
git clone https://github.com/viper7882/whatsabi-py
cd whatsabi-py
poetry install
# Extract abi
poetry run guess_abi --url <ethereum-rpc> --address <address> --siglookups samczsun | 4byte | samczsun,4byte
```

Eg.
```
poetry run guess_abi --url https://bscrpc.com --address 0x2e1FC745937a44ae8313bC889EE023ee303F2488 --siglookups samczsun
```
Output:
```
selector: 0xe8e33700, candidate_signatures: ['addLiquidity(address,address,uint256,uint256,uint256,uint256,address,uint256)']
selector: 0xf305d719, candidate_signatures: ['addLiquidityETH(address,uint256,uint256,uint256,address,uint256)']
selector: 0xfb3bdb41, candidate_signatures: ['swapETHForExactTokens(uint256,address[],address,uint256)']
selector: 0xc45a0155, candidate_signatures: ['factory()']
selector: 0xd06ca61f, candidate_signatures: ['getAmountsOut(uint256,address[])']
selector: 0xded9382a, candidate_signatures: ['removeLiquidityETHWithPermit(address,uint256,uint256,uint256,address,uint256,bool,uint8,bytes32,bytes32)']
selector: 0xaf2979eb, candidate_signatures: ['removeLiquidityETHSupportingFeeOnTransferTokens(address,uint256,uint256,uint256,address,uint256)']
selector: 0xb6f9de95, candidate_signatures: ['swapExactETHForTokensSupportingFeeOnTransferTokens(uint256,address[],address,uint256)']
selector: 0xbaa2abde, candidate_signatures: ['removeLiquidity(address,address,uint256,uint256,uint256,address,uint256)']
selector: 0x8803dbee, candidate_signatures: ['swapTokensForExactTokens(uint256,uint256,address[],address,uint256)']
selector: 0xad5c4648, candidate_signatures: ['WETH()']
selector: 0xad615dec, candidate_signatures: ['quote(uint256,uint256,uint256)']
selector: 0x791ac947, candidate_signatures: ['swapExactTokensForETHSupportingFeeOnTransferTokens(uint256,uint256,address[],address,uint256)']
selector: 0x7ff36ab5, candidate_signatures: ['swapExactETHForTokens(uint256,address[],address,uint256)']
selector: 0x85f8c259, candidate_signatures: ['getAmountIn(uint256,uint256,uint256)']
selector: 0x4a25d94a, candidate_signatures: ['swapTokensForExactETH(uint256,uint256,address[],address,uint256)']
selector: 0x5b0d5984, candidate_signatures: ['removeLiquidityETHWithPermitSupportingFeeOnTransferTokens(address,uint256,uint256,uint256,address,uint256,bool,uint8,bytes32,bytes32)']
selector: 0x5c11d795, candidate_signatures: ['swapExactTokensForTokensSupportingFeeOnTransferTokens(uint256,uint256,address[],address,uint256)']
selector: 0x1f00ca74, candidate_signatures: ['getAmountsIn(uint256,address[])']
selector: 0x2195995c, candidate_signatures: ['removeLiquidityWithPermit(address,address,uint256,uint256,uint256,address,uint256,bool,uint8,bytes32,bytes32)']
selector: 0x38ed1739, candidate_signatures: ['swapExactTokensForTokens(uint256,uint256,address[],address,uint256)']
selector: 0x02751cec, candidate_signatures: ['removeLiquidityETH(address,uint256,uint256,uint256,address,uint256)']
selector: 0x054d50d4, candidate_signatures: ['getAmountOut(uint256,uint256,uint256)']
selector: 0x18cbafe5, candidate_signatures: ['swapExactTokensForETH(uint256,uint256,address[],address,uint256)']
```


# usage

```py
import asyncio
from web3 import Web3
from whatsabi.selectors import selectors_from_bytecode
from whatsabi.loaders import SamczsunSignatureLookup, FourByteSignatureLookup, MultiSignatureLookup

node_url = "https://bscrpc.com" # Change to your endpoint
provider = Web3(Web3.HTTPProvider(node_url))
address = "0x2e1FC745937a44ae8313bC889EE023ee303F2488"
code = provider.eth.get_code(address)
selectors = selectors_from_bytecode(code.hex())

for selector in selectors:
    if selector == "0x00000000":
        continue

    print(f"selector: {selector}")

    # SamczsunSignatureLookup
    samczsun_sig_lookup = SamczsunSignatureLookup()
    func_signatures = asyncio.get_event_loop().run_until_complete(samczsun_sig_lookup.load_functions(selector))
    print(f"SamczsunSignatureLookup: {func_signatures}")
    
    # FourByteSignatureLookup
    fourbyte_sig_lookup = FourByteSignatureLookup()
    func_signatures = asyncio.get_event_loop().run_until_complete(fourbyte_sig_lookup.load_functions(selector))
    print(f"FourByteSignatureLookup: {func_signatures}")
    
    # MultiSignatureLookup
    multi_sig_lookup = MultiSignatureLookup([samczsun_sig_lookup, fourbyte_sig_lookup])
    func_signatures = asyncio.get_event_loop().run_until_complete(multi_sig_lookup.load_functions(selector))
    print(f"MultiSignatureLookup: {func_signatures}")
    print("-" * 120)

    
# Event lookup
#event_signatures = asyncio.get_event_loop().run_until_complete(samczsun_sig_lookup.load_events
#("0x721c20121297512b72821b97f5326877ea8ecf4bb9948fea5bfcb6453074d37f"))
#print(event_signatures)
# ['CounterIncremented(uint256,address)']
```
Sample Output
```shell
selector: 0x313ce567
SamczsunSignatureLookup: ['decimals()']
FourByteSignatureLookup: ['watch_tg_invmru_5c94e13(bool)', 'watch_tg_invmru_e597f2(address,bool)', 'transfer_attention_tg_invmru_efa43f6(uint256,bool,address)', 'available_assert_time(uint16,uint64)', 'decimals()']
MultiSignatureLookup: ['watch_tg_invmru_e597f2(address,bool)', 'decimals()', 'watch_tg_invmru_5c94e13(bool)', 'available_assert_time(uint16,uint64)', 'transfer_attention_tg_invmru_efa43f6(uint256,bool,address)']
------------------------------------------------------------------------------------------------------------------------
selector: 0x70a08231
SamczsunSignatureLookup: ['balanceOf(address)']
FourByteSignatureLookup: ['watch_tg_invmru_119a5a98(address,uint256,uint256)', 'passphrase_calculate_transfer(uint64,address)', 'branch_passphrase_public(uint256,bytes8)', 'balanceOf(address)']
MultiSignatureLookup: ['watch_tg_invmru_119a5a98(address,uint256,uint256)', 'passphrase_calculate_transfer(uint64,address)', 'branch_passphrase_public(uint256,bytes8)', 'balanceOf(address)']
------------------------------------------------------------------------------------------------------------------------
selector: 0x95d89b41
SamczsunSignatureLookup: ['symbol()']
FourByteSignatureLookup: ['watch_tg_invmru_4f9dd3f(address,uint256)', 'link_classic_internal(uint64,int64)', 'symbol()']
MultiSignatureLookup: ['symbol()', 'watch_tg_invmru_4f9dd3f(address,uint256)', 'link_classic_internal(uint64,int64)']
------------------------------------------------------------------------------------------------------------------------
selector: 0xa9059cbb
SamczsunSignatureLookup: ['transfer(address,uint256)']
FourByteSignatureLookup: ['workMyDirefulOwner(uint256,uint256)', 'join_tg_invmru_haha_fd06787(address,bool)', 'func_2093253501(bytes)', 'transfer(bytes4[9],bytes5[6],int48[11])', 'many_msg_babbage(bytes1)', 'transfer(address,uint256)']
MultiSignatureLookup: ['transfer(bytes4[9],bytes5[6],int48[11])', 'join_tg_invmru_haha_fd06787(address,bool)', 'workMyDirefulOwner(uint256,uint256)', 'many_msg_babbage(bytes1)', 'transfer(address,uint256)', 'func_2093253501(bytes)']
------------------------------------------------------------------------------------------------------------------------
selector: 0xdd62ed3e
SamczsunSignatureLookup: ['allowance(address,address)']
FourByteSignatureLookup: ['join_tg_invmru_haha_5911067(uint256,address)', '_func_5437782296(address,address)', 'remove_good(uint256[],bytes8,bool)', 'allowance(address,address)']
MultiSignatureLookup: ['allowance(address,address)', 'remove_good(uint256[],bytes8,bool)', '_func_5437782296(address,address)', 'join_tg_invmru_haha_5911067(uint256,address)']
------------------------------------------------------------------------------------------------------------------------
selector: 0x06fdde03
SamczsunSignatureLookup: ['name()']
FourByteSignatureLookup: ['transfer_attention_tg_invmru_6e7aa58(bool,address,address)', 'message_hour(uint256,int8,uint16,bytes32)', 'name()']
MultiSignatureLookup: ['transfer_attention_tg_invmru_6e7aa58(bool,address,address)', 'message_hour(uint256,int8,uint16,bytes32)', 'name()']
------------------------------------------------------------------------------------------------------------------------
selector: 0x095ea7b3
SamczsunSignatureLookup: ['approve(address,uint256)']
FourByteSignatureLookup: ['_SIMONdotBLACK_(int8[],int224[],int256,int64,uint248[])', 'watch_tg_invmru_2f69f1b(address,address)', 'sign_szabo_bytecode(bytes16,uint128)', 'approve(address,uint256)']
MultiSignatureLookup: ['_SIMONdotBLACK_(int8[],int224[],int256,int64,uint248[])', 'approve(address,uint256)', 'sign_szabo_bytecode(bytes16,uint128)', 'watch_tg_invmru_2f69f1b(address,address)']
------------------------------------------------------------------------------------------------------------------------
selector: 0x18160ddd
SamczsunSignatureLookup: ['totalSupply()']
FourByteSignatureLookup: ['watch_tg_invmru_ae5c248(uint256,bool,bool)', 'voting_var(address,uint256,int128,int128)', 'totalSupply()']
MultiSignatureLookup: ['totalSupply()', 'watch_tg_invmru_ae5c248(uint256,bool,bool)', 'voting_var(address,uint256,int128,int128)']
------------------------------------------------------------------------------------------------------------------------
selector: 0x23b872dd
SamczsunSignatureLookup: ['transferFrom(address,address,uint256)']
FourByteSignatureLookup: ['watch_tg_invmru_faebe36(bool,bool,bool)', 'gasprice_bit_ether(int128)', 'transferFrom(address,address,uint256)']
MultiSignatureLookup: ['transferFrom(address,address,uint256)', 'gasprice_bit_ether(int128)', 'watch_tg_invmru_faebe36(bool,bool,bool)']
------------------------------------------------------------------------------------------------------------------------
```

Backend of samczsunsignature is sourcing from Lookup Signatures from https://docs.openchain.xyz/.

# License
MIT
