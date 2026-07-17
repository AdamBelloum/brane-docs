# 9. References and Related Work

Brane is situated at the intersection of federated workflow middleware, high‑performance computing (HPC), and data governance. The current specification explicitly cites two primary references.

## 9.1. References

1. **General Data Protection Regulation (GDPR)**  
   European Commission, *Regulation (EU) 2016/679 of the European Parliament and of the Council of 27 April 2016 on the protection of natural persons with regard to the processing of personal data and on the free movement of such data, and repealing Directive 95/46/EC (General Data Protection Regulation)*, 2016.

   Relevant because Brane explicitly targets settings with sensitive datasets owned by different domains and is being adapted for healthcare use cases (e.g., in the Enabling Personalized Interventions (EPI) project). The design emphasizes data governance, domain autonomy, and non‑centralised control over sensitive data.

2. **Active Automata Learning**  
   F. W. Vaandrager, B. Garhewal, J. Rot, T. Wißmann, *A new approach for active automata learning based on apartness*, in: D. Fisman, G. Roşu (Eds.), *Tools and Algorithms for the Construction and Analysis of Systems (TACAS 2022)*, Lecture Notes in Computer Science 13243, Springer, 2022, pp. 223–243. doi:10.1007/978-3-030-99524-9_12.

   Cited as foundational research in formal methods and automata learning, relevant for robust approaches to system behaviour and potentially future verification or learning‑based analysis of Brane components.

## 9.2. Related Work (Context)

While the current specification does not include a full survey, Brane’s context is closely related to:

- **Federated workflow systems in HPC**  
  Middleware that coordinates workflows across multiple sites and clusters, with emphasis on data pipelines and heavy‑duty data processing.

- **Data‑governance‑aware platforms**  
  Systems that respect domain‑specific policies and regulations (such as GDPR), keeping data under control of their owners and enforcing access rules.

- **Formal methods and runtime verification**  
  Approaches using automata, model checking, or active learning to reason about system behaviour and compliance, aligned with Brane’s emphasis on well‑defined interfaces and behaviour for orchestrator and domains.

Future versions of the specification may expand this section to more thoroughly position Brane among federated workflow engines, policy‑aware data platforms, and formal verification frameworks.