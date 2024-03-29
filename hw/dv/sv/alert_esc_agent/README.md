# ALERT_ESC Agent

ALERT_ESC UVM Agent is extended from DV library agent classes.

## Description

This agent implements both alert(alert_rx, alert_tx) and escalation (esc_rx,
esc_tx) interface protocols, and can be configured to behave in both host and
device modes. For design documentation, please refer to [alert_handler
spec](../../../top_earlgrey/ip_autogen/alert_handler/README.md).

### Alert Agent

Alert agent supports both synchronous and asynchronous modes.

#### Alert Device Agent

For IPs that send out alerts, it is recommended to attach alert device agents to
the block level testbench.
Please refer to [cip_lib documentation](../cip_lib/README.md)
regarding instructions to configure alert device agent in DV testbenches.

### Escalation Agent
