├── .gitignore
├── FundMe.sol
├── LICENSE
└── PriceConverter.sol


/.gitignore:
--------------------------------------------------------------------------------
 1 | # Remix compiler artifacts
 2 | **/artifacts/
 3 | **/artifacts/**
 4 | 
 5 | # Remix plugin state folders
 6 | deps/
 7 | states/
 8 | 
 9 | # Debug info
10 | *.dbg.json
11 | *.tsbuildinfo
12 | 
13 | # Optional
14 | .env
15 | .env.local


--------------------------------------------------------------------------------
/FundMe.sol:
--------------------------------------------------------------------------------
 1 | // SPDX-License-Identifier: MIT
 2 | pragma solidity ^0.8.18;
 3 | 
 4 | import {AggregatorV3Interface} from "@chainlink/contracts/src/v0.8/shared/interfaces/AggregatorV3Interface.sol";
 5 | import {PriceConverter} from "./PriceConverter.sol";
 6 | 
 7 | error NotOwner();
 8 | 
 9 | contract FundMe {
10 |     using PriceConverter for uint256;
11 | 
12 |     mapping(address => uint256) public addressToAmountFunded;
13 |     address[] public funders;
14 | 
15 |     // Could we make this constant?  /* hint: no! We should make it immutable! */
16 |     address public /* immutable */ i_owner;
17 |     uint256 public constant MINIMUM_USD = 5 * 10 ** 18;
18 | 
19 |     constructor() {
20 |         i_owner = msg.sender;
21 |     }
22 | 
23 |     function fund() public payable {
24 |         require(msg.value.getConversionRate() >= MINIMUM_USD, "You need to spend more ETH!");
25 |         // require(PriceConverter.getConversionRate(msg.value) >= MINIMUM_USD, "You need to spend more ETH!");
26 |         addressToAmountFunded[msg.sender] += msg.value;
27 |         funders.push(msg.sender);
28 |     }
29 | 
30 |     function getVersion() public view returns (uint256) {
31 |         AggregatorV3Interface priceFeed = AggregatorV3Interface(0x694AA1769357215DE4FAC081bf1f309aDC325306);
32 |         return priceFeed.version();
33 |     }
34 | 
35 |     modifier onlyOwner() {
36 |         // require(msg.sender == owner);
37 |         if (msg.sender != i_owner) revert NotOwner();
38 |         _;
39 |     }
40 | 
41 |     function withdraw() public onlyOwner {
42 |         for (uint256 funderIndex = 0; funderIndex < funders.length; funderIndex++) {
43 |             address funder = funders[funderIndex];
44 |             addressToAmountFunded[funder] = 0;
45 |         }
46 |         funders = new address[](0);
47 |         
48 |         (bool callSuccess,) = payable(msg.sender).call{value: address(this).balance}("");
49 |         require(callSuccess, "Call failed");
50 |     }
51 | 
52 |     fallback() external payable {
53 |         fund();
54 |     }
55 | 
56 |     receive() external payable {
57 |         fund();
58 |     }
59 | }
60 | 
61 | 


--------------------------------------------------------------------------------
/LICENSE:
--------------------------------------------------------------------------------
 1 | MIT License
 2 | 
 3 | Copyright (c) 2025 Nachiketa
 4 | 
 5 | Permission is hereby granted, free of charge, to any person obtaining a copy
 6 | of this software and associated documentation files (the "Software"), to deal
 7 | in the Software without restriction, including without limitation the rights
 8 | to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 9 | copies of the Software, and to permit persons to whom the Software is
10 | furnished to do so, subject to the following conditions:
11 | 
12 | The above copyright notice and this permission notice shall be included in all
13 | copies or substantial portions of the Software.
14 | 
15 | THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
16 | IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
17 | FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
18 | AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
19 | LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
20 | OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
21 | SOFTWARE.
22 | 


--------------------------------------------------------------------------------
/PriceConverter.sol:
--------------------------------------------------------------------------------
 1 | // SPDX-License-Identifier: MIT
 2 | pragma solidity ^0.8.18;
 3 | 
 4 | import {AggregatorV3Interface} from "@chainlink/contracts/src/v0.8/shared/interfaces/AggregatorV3Interface.sol"; 
 5 | 
 6 | library PriceConverter {
 7 |     function getPrice() internal view returns (uint256) {
 8 |         AggregatorV3Interface priceFeed = AggregatorV3Interface(
 9 |             0x694AA1769357215DE4FAC081bf1f309aDC325306
10 |         );
11 |         (, int256 answer, , , ) = priceFeed.latestRoundData();
12 |         // ETH/USD rate in 18 digit
13 |         return uint256(answer * 10000000000);
14 |     }
15 | 
16 |     // 1000000000
17 |     function getConversionRate(
18 |         uint256 ethAmount
19 |     ) internal view returns (uint256) {
20 |         uint256 ethPrice = getPrice();
21 |         uint256 ethAmountInUsd = (ethPrice * ethAmount) / 1000000000000000000;
22 |         return ethAmountInUsd;
23 |     }
24 | }


--------------------------------------------------------------------------------
