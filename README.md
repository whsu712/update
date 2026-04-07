# update
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.34;

contract BaseLuckyNumber {
    address public immutable owner;
    uint256 public treasuryBalance;

    event NumberGuessed(
        address indexed player,
        uint8 guessedNumber,
        uint8 luckyNumber,
        uint256 bet,
        uint256 payout,
        uint256 timestamp
    );

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can call this");
        _;
    }

    function guess(uint8 _number) external payable {
        require(msg.value == 0.001 ether, "Must pay exactly 0.001 ETH");
        require(_number >= 1 && _number <= 100, "Number must be between 1 and 100");

        // 生成幸运数字 1~100
        uint256 random = uint256(keccak256(abi.encodePacked(
            block.prevrandao,
            msg.sender,
            block.timestamp,
            block.number
        ))) % 100 + 1;

        uint8 luckyNumber = uint8(random);
        uint256 payout = 0;

        if (_number == luckyNumber) {
            payout = 0.008 ether;           // 8x big lotter
            payable(msg.sender).transfer(payout);
        } else {
            treasuryBalance += msg.value;   // false
        }

        emit NumberGuessed(msg.sender, _number, luckyNumber, msg.value, payout, block.timestamp);
    }

    function getTreasury() external view returns (uint256) {
        return treasuryBalance;
    }

    function withdrawTreasury() external onlyOwner {
        uint256 amount = treasuryBalance;
        treasuryBalance = 0;
        payable(owner).transfer(amount);
    }

    function getContractInfo() external pure returns (string memory) {
        return "BaseLuckyNumber - Pay 0.001 ETH to guess 1-100 and win 8x!";
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract BaseToken is ERC20, Ownable {
    constructor() ERC20("BaseToken", "BTK") Ownable(msg.sender) {
        _mint(msg.sender, 1000000000 * 10 ** decimals());
    }

    function mint(address to, uint256 amount) public onlyOwner {
        _mint(to, amount);
    }
}
