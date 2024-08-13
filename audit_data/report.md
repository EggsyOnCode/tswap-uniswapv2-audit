### [M-#] The invariant condition of the AMM breaking because of rebasing / fee on transfer feature in the protocol

**Description**
In the `TSwapPool:_swap` we maintain a global swap_count which if reached some MAX_COUNT sends 1e18 output token to the msg.sender messing up the constant product market maker equivalence. Hence, breaking our invariance.

**Impact**
`HIGH` since this can be exploited to manipulate the prices of tokens in the pool

**Proof of Concepts**

```javascript

contract Invariant is StdInvariant, Test {
    PoolFactory factory;
    TSwapPool pool;
    ERC20Mock poolToken;
    ERC20Mock weth;
    ERC20Mock tokenB;

    int256 constant STARTING_X = 100e18; // starting ERC20
    int256 constant STARTING_Y = 50e18; // starting WETH
    uint256 constant FEE = 997e15; //
    int256 constant MATH_PRECISION = 1e18;

    TSwapPoolHandler handler;

    function setUp() public {
        weth = new ERC20Mock();
        poolToken = new ERC20Mock();
        factory = new PoolFactory(address(weth));
        pool = TSwapPool(factory.createPool(address(poolToken)));

        // Create the initial x & y values for the pool
        poolToken.mint(address(this), uint256(STARTING_X));
        weth.mint(address(this), uint256(STARTING_Y));
        poolToken.approve(address(pool), type(uint256).max);
        weth.approve(address(pool), type(uint256).max);
        pool.deposit(uint256(STARTING_Y), uint256(STARTING_Y), uint256(STARTING_X), uint64(block.timestamp));

        handler = new TSwapPoolHandler(pool);

        bytes4[] memory selectors = new bytes4[](2);
        selectors[0] = TSwapPoolHandler.deposit.selector;
        selectors[1] = TSwapPoolHandler.swapPoolTokenForWethBasedOnOutputWeth.selector;

        targetSelector(FuzzSelector({ addr: address(handler), selectors: selectors }));
        targetContract(address(handler));
    }

    function invariant_deltaXFollowsMath() public {
        assertEq(handler.actualDeltaX(), handler.expectedDeltaX());
    }

    function invariant_deltaYFollowsMath() public {
        assertEq(handler.actualDeltaY(), handler.expectedDeltaY());
    }
}


```

The last tests fails. 

**Recommended mitigation**

Remove the logic rebasing logic from the contract
