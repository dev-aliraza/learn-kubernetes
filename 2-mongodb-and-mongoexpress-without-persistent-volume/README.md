# MongoDB and Mongo Express Setup without Persistent Volume
- Calico has been used for overlay networking between nodes and pods
- MetalLB (without speaker) has been used to assign external IP for load balancer(s)
- Calico CLI tool has been used to create `BGPConfiguration` using `calico-bgp-config.yaml`
- MetalLB address pool has been added manually using `metallb-bgp-address-pool.yaml`