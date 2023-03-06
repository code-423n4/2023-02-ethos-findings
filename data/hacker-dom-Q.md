## ReaperStrategyGranarySupplyOnly lacks appropriate logging

`ReaperStrategyGranarySupplyOnly` nor either of its base (parent) contracts, `ReaperBaseStrategyv4` and `VeloSolidMixin`, emit any events for important operations. This can make it difficult to monitor the contract and make sense of the history of its interaction. This is especially important in the event of a security breach. 