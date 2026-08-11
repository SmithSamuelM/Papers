# New Default logic for Unary Edge Operators

The orginal uses cases for edges in ACDCs were to model delegation chains of authority. The run up to SEDI has produced other important use cases that are not delegative in nature. The existing global default rules for Unary edge operators feels cumbersome in those applications. 

This discussion proposes some changes to the Unary edge operator defaulting logic.
The main advantage of defaults is compactness of expression. A set of edges by omitting an explicit edge operator benefits from an intelligent default. This makes the expression of edges less cumbersome.
However then there is no intelligent default then all edges must have explicit operators. This makes it cumbersome.
So the idea is to discuss how to have more fine grained defualt logic to better manage this trade-off.

## Current Unary Edge Operators and Defaults

