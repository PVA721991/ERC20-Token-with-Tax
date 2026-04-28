# ERC20-Token-with-Tax
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

contract TaxableToken {
    string public name = "Base Tax Token";
    string public symbol = "BTT";
    uint8 public decimals = 18;
    uint256 public totalSupply;

    uint256 public buyTax = 500;  // 5%
    uint256 public sellTax = 500; // 5%
    address public taxWallet;

    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;

    error InsufficientBalance();

    event Transfer(address indexed from, address indexed to, uint256 value);
    event TaxCollected(uint256 amount);

    constructor(uint256 initialSupply, address _taxWallet) {
        totalSupply = initialSupply * 10 ** decimals;
        balanceOf[msg.sender] = totalSupply;
        taxWallet = _taxWallet;
        emit Transfer(address(0), msg.sender, totalSupply);
    }

    function transfer(address to, uint256 amount) public returns (bool) {
        if (balanceOf[msg.sender] < amount) revert InsufficientBalance();

        uint256 taxAmount = (amount * buyTax) / 10000;
        uint256 sendAmount = amount - taxAmount;

        balanceOf[msg.sender] -= amount;
        balanceOf[to] += sendAmount;
        balanceOf[taxWallet] += taxAmount;

        emit Transfer(msg.sender, to, sendAmount);
        emit Transfer(msg.sender, taxWallet, taxAmount);
        return true;
    }
}
