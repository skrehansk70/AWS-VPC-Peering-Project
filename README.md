# AWS VPC Peering Project

## Project Overview
This project demonstrates private communication between two Amazon Virtual Private Clouds (VPCs) using an AWS VPC Peering connection. A Bastion Host in VPC A is used as the SSH entry point for the private EC2 instances.

## Objectives
- Create two VPCs with separate IPv4 CIDR ranges.
- Configure public/private subnets and route tables.
- Establish and activate a VPC Peering connection.
- Configure security groups to control SSH and ICMP traffic.
- Connect to private EC2 instances through a Bastion Host.
- Verify communication in both directions.
- Demonstrate controlled failure and recovery by temporarily removing and restoring an ICMP rule.

## Architecture

```text
                     Internet
                         |
                  Internet Gateway
                         |
                 VPC A: 10.10.0.0/16
                         |
                  Public Subnet
                         |
                 Bastion Host
                  10.10.1.10
                         |
             -------------------------
             |                       |
       Private Subnet A         VPC Peering
       10.10.2.0/24          (Active connection)
             |                       |
          App-A              ----------------
       10.10.2.10                     |
                              VPC B: 10.20.0.0/16
                                      |
                               Private Subnet B
                                10.20.1.0/24
                                      |
                                    App-B
                                 10.20.1.10
```

> Update this diagram if your final AWS resource configuration or subnet CIDRs differ from the values shown above.

## Resource Summary

| Resource | Configuration |
|---|---|
| AWS Region | Asia Pacific (Mumbai), `ap-south-1` |
| VPC A CIDR | `10.10.0.0/16` |
| VPC A private subnet | `10.10.2.0/24` |
| Bastion Host private IP | `10.10.1.10` |
| App-A private IP | `10.10.2.10` |
| VPC B CIDR | `10.20.0.0/16` |
| VPC B private subnet | `10.20.1.0/24` |
| App-B private IP | `10.20.1.10` |
| VPC Peering status | Active (verify with your final screenshot) |

## Routing Configuration

The route tables should include:
- VPC A private route table: local route for `10.10.0.0/16` and a peering route for `10.20.0.0/16`.
- VPC B private route table: local route for `10.20.0.0/16` and a peering route for `10.10.0.0/16`.
- VPC A public route table: local route, an Internet Gateway route for `0.0.0.0/0`, and (if configured in this project) a route for `10.20.0.0/16` through the peering connection.

Confirm the actual route tables and subnet associations in the AWS Console before submitting.

## Security Groups

The project uses security groups to restrict access:
- Bastion Host: SSH (TCP 22) allowed only from the administrator's current public IP address (`/32`).
- App-A: SSH access from the Bastion Host and ICMP Echo Request from App-B.
- App-B: SSH access from the Bastion Host and ICMP Echo Request from App-A.

Use the actual security group names and rules shown in your AWS Console. Do not open SSH to `0.0.0.0/0`.

## Testing and Results

### Test 1: App-A to App-B
Command run on App-A:

```bash
ping -c 4 10.20.1.10
```

Observed result: 4 packets transmitted, 4 received, 0% packet loss.

### Test 2: App-B to App-A
Command run on App-B:

```bash
ping -c 4 10.10.2.10
```

Observed result: 4 packets transmitted, 4 received, 0% packet loss.

### Test 3: Controlled Failure
The ICMP Echo Request inbound rule allowing traffic from App-A was temporarily removed from App-B's security group.

Command run on App-A:

```bash
ping -c 4 10.20.1.10
```

Observed result: 4 packets transmitted, 0 received, 100% packet loss.

### Test 4: Recovery
The ICMP rule was restored and the ping test was repeated.

Observed result: 4 packets transmitted, 4 received, 0% packet loss.

> Replace or supplement these summaries with your own screenshots in the `screenshots/` folder.

## Evidence Screenshots

Add your screenshots to the `screenshots/` folder. Use the checklist in [`screenshots/README.md`](screenshots/README.md).

## Key Learnings
- VPC Peering enables private routing between the two VPC CIDR ranges when the required routes exist.
- Both sides need appropriate routes for two-way communication.
- Security groups must allow the intended traffic in addition to route-table configuration.
- A Bastion Host provides an SSH entry point to private instances.
- VPC Peering is not a substitute for an Internet Gateway or NAT Gateway for internet access.

## Cleanup
After collecting evidence and completing the report, terminate only the EC2 instances created for this project and remove the associated project resources if they are no longer required. Verify resource associations before deleting anything, and do not delete unrelated AWS resources.

## Submission Checklist
- [ ] Confirm the resource configuration and update any values that differ from the table above.
- [ ] Upload evidence screenshots.
- [ ] Review the repository for sensitive information.
- [ ] Add the final project report PDF if required by the trainer.
- [ ] Submit the GitHub repository URL through the Google Form when it is shared.

## Security Note
Never commit private keys (`.pem`), AWS access keys, passwords, session tokens, or other credentials to GitHub.

**Note:** All project evidence screenshots, including AWS VPC configurations, subnet settings, VPC peering status, route tables, EC2 instances, connectivity tests, controlled failure, and recovery, are available in the [`screenshots/README.md`](screenshots/README.md) file.
