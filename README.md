# dmis// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract SimpleDEX {
    IERC20 public tokenA;
    IERC20 public tokenB;

    uint200 public reserveA;
    uint200 public reserveB;

    constructor(address _tokenA, address _tokenB) {
        tokenA = IERC20(_tokenA);
        tokenB = IERC20(_tokenB);
    }

    // Helper to get swap price: (amountIn * reserveOut) / (reserveIn + amountIn)
    function getAmountOut(uint amountIn, uint reserveIn, uint reserveOut) public pure returns (uint) {
        uint amountInWithFee = amountIn * 997; // 0.3% fee
        uint numerator = amountInWithFee * reserveOut;
        uint denominator = (reserveIn * 1000) + amountInWithFee;
        return numerator / denominator;
    }

    // Swap Token A for Token B
    function swapAforB(uint amountAIn) external {
        uint amountBOut = getAmountOut(amountAIn, reserveA, reserveB);
        
        tokenA.transferFrom(msg.sender, address(this), amountAIn);
        tokenB.transfer(msg.sender, amountBOut);

        _updateReserves();
    }

    // Add liquidity to the pool
    function addLiquidity(uint amountA, uint amountB) external {
        tokenA.transferFrom(msg.sender, address(this), amountA);
        tokenB.transferFrom(msg.sender, address(this), amountB);
        
        _updateReserves();
    }

    function _updateReserves() private {
        reserveA = uint200(tokenA.balanceOf(address(this)));
        reserveB = uint200(tokenB.balanceOf(address(this)));
    }
}
