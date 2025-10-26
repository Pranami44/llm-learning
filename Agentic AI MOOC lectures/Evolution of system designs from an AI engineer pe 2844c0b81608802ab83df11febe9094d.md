# Evolution of system designs from an AI engineer perspective

**Main Takeaways:** This lecture presents a practitioner's perspective on how AI infrastructure has evolved to become the third essential pillar of IT strategy, alongside traditional computing and networking. It traces how successive algorithmic innovations continue to drive LLM progress while consumer applications remain fluid and enterprise adoption accelerates.

It advocates for building an "AI cloud" designed for exaflops-scale, heterogeneous, cloud-native compute that supports workloads beyond just training. The talk emphasizes practical engineering realities: hardware failures are inevitable, inference scaling presents new cost and capacity challenges, and successful deployment requires AI-native platforms that unify development, training, and inference across multi-cloud environments.

# **Key Points:**

## **Algorithmic Waves Driving Progress**

- Continuous improvements in model architectures and training techniques propel LLM capabilities forward, creating new system design requirements with each generation

![Screenshot 2025-10-06 at 6.06.42 AM.png](Evolution%20of%20system%20designs%20from%20an%20AI%20engineer%20pe%202844c0b81608802ab83df11febe9094d/Screenshot_2025-10-06_at_6.06.42_AM.png)

## **Divergent Market Dynamics**

- Consumer applications are still exploring product-market fit while enterprise adoption accelerates rapidly, creating different infrastructure demands

## **AI as Third IT Pillar**

- AI infrastructure now stands alongside compute and networking as a fundamental IT strategy component, requiring dedicated investment and architecture

## **AI Cloud Architecture**

- Purpose-built infrastructure for exaflops-scale, heterogeneous computing that handles diverse workloads beyond training alone

![Screenshot 2025-10-06 at 6.59.53 AM.png](Evolution%20of%20system%20designs%20from%20an%20AI%20engineer%20pe%202844c0b81608802ab83df11febe9094d/Screenshot_2025-10-06_at_6.59.53_AM.png)

### Best practices for a lot of the AI startups

- Find a multiple cloud supply chain
- Elasticity and utilization management
- AI native platform to unify dev, training and inferance
- Build you own team around model and applications

## **Hardware Failure Reality**

- GPUs and accelerators fail regularly; systems must be designed with redundancy and graceful degradation from the start

## **Inference Scaling Challenges**

- Growing inference demands require careful capacity planning and cost management strategies different from training workloads

## **Multi-Cloud Supply Chain**

- Adopting multiple cloud providers ensures resilience and flexibility in accessing scarce compute resources

## **AI-Native Platform Requirements**

- Successful systems need unified platforms that seamlessly integrate development, training, and inference workflows

## **Developer Efficiency vs. Reliability**

- Balancing rapid iteration needs with production stability requirements in agentic system design