  `` uint constant public SECONDS_IN_ONE_MINUTE = 60; `` is redundant, uses storage and costs gas, 
I suggest using ``60 seconds`` directly in the functions as ``seconds`` is a valid synthax in solidity.
	