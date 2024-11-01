## Variables for the configurations
NS1 = ns1
NS2 = ns2
VETH1 = veth1
VETH2 = veth2
IP1 = 10.0.0.1/24
IP2 = 10.0.0.2/24
BRIDGE_NAME = br0
CURRENT_DIR := $(pwd)
LOG_DIR = "./logs"
TCPDUMP_LOG = $(LOG_DIR)/capture.pcap

.PHONY: setup create_logs create_namespaces create_interfaces assigning_interfaces configure_interfaces status clean capture-ns1 capture-ns2 test-ns1 test-ns2

setup: create_logs create_namespaces create_interfaces assigning_interfaces configure_interfaces status

create_logs:
	@echo "Creating logs directory at $(LOG_DIR)..."
	@cd $(CURRENT_DIR)
	@mkdir -p $(LOG_DIR)
	@touch $(TCPDUMP_LOG)
	@echo "Logs directory created successfully."

# creating network namespaces
create_namespaces:
	@echo "creating network namespaces..."
	sudo ip netns add $(NS1)
	sudo ip netns add $(NS2)
	@echo "network namespaces created successfully."

create_interfaces:
	@echo "creating virtual interfaces"
	sudo ip link add $(VETH1) type veth peer name $(VETH2)
	@echo "creation of interfaces successful"

assigning_interfaces:
	@echo "assigning interfaces"
	sudo ip link set $(VETH1) netns $(NS1)
	sudo ip link set $(VETH2) netns $(NS2)
	@echo "assigning successful"

configure_interfaces:
	@echo "Configuring interfaces..."
	# Configure NS1
	sudo ip netns exec $(NS1) ip addr add $(IP1) dev $(VETH1)
	sudo ip netns exec $(NS1) ip link set $(VETH1) up
	sudo ip netns exec $(NS1) ip link set lo up
	
	# Configure NS2
	sudo ip netns exec $(NS2) ip addr add $(IP2) dev $(VETH2)
	sudo ip netns exec $(NS2) ip link set $(VETH2) up
	sudo ip netns exec $(NS2) ip link set lo up
	@echo "Interface configuration completed."

capture-ns1:
	@echo "Starting packet capture on $(NS1)..."
	-sudo ip netns exec $(NS1) tcpdump -i $(VETH1) -w $(TCPDUMP_LOG) &
	@echo "Packet capture started. Log file: $(TCPDUMP_LOG)"

capture-ns2:
	@echo "Starting packet capture on $(NS2)..."
	-sudo ip netns exec $(NS2) tcpdump -i $(VETH2) -w $(TCPDUMP_LOG)
	@echo "Packet capture started. Log file: $(TCPDUMP_LOG)"

test-ns1:
	@echo "Testing connectivity from $(NS1)..."
	@echo "Pinging from $(NS1) to $(NS2)..."
	sudo ip netns exec $(NS1) ping -c 4 10.0.0.2
	@echo "Network test completed."

test-ns2:
	@echo "Testing connectivity from $(NS2)..."
	@echo "Pinging from $(NS2) to $(NS1)..."
	sudo ip netns exec $(NS2) ping -c 4 10.0.0.1
	@echo "Network test completed."

clean:
	@echo "Cleaning up environment..."
	-sudo ip netns del $(NS1) 2>/dev/null
	-sudo ip netns del $(NS2) 2>/dev/null
	-sudo rm -rf $(LOG_DIR)
	@echo "Cleanup completed."

status:
	@echo "Network Namespace Status:"
	@echo "========================"
	@echo "Logs directory: $(LOG_DIR)"
	@echo "\nListing network namespaces:"
	sudo ip netns list
	@echo "\nInterface status in $(NS1):"
	-sudo ip netns exec $(NS1) ip addr show
	@echo "\nInterface status in $(NS2):"
	-sudo ip netns exec $(NS2) ip addr show