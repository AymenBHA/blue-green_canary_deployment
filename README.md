# blue-green_canary_deployment (AWS)
Blue-green and canary
In modern cloud environments, deploying new versions of an application shouldn't mean taking your system offline. This project builds a highly resilient infrastructure by deploying two identical, fully isolated environments (Blue and Green) behind an AWS Application Load Balancer (ALB).

we insure transition traffic seamlessly from your live environment (Version 1 / Blue) to our updated environment (Version 2 / Green) using a Canary deployment strategy. Rather than flipping a switch and hoping for the best, a Canary strategy allows us to route a small percentage of user traffic (e.g., 20%) to the new version to monitor for errors before committing to a full 100% cutover. This ensures zero downtime and minimizes the blast radius if an update fails.

