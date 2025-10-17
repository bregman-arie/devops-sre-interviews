# Kubernetes - Advanced Interview Mock #2

> **Difficulty:** Advanced  
> **Duration:** ~30 minutes (single deep-dive)  
> **Goal:** Evaluate systematic production incident triage and Kubernetes ingress / networking expertise.

---

## 🧠 Deep-Dive Question

You’ve deployed an Nginx ingress controller, but some requests hang indefinitely while others succeed.

What steps would you take to debug and isolate the cause?

[📖 Answer](mock_2_answers.md#youve-deployed-an-nginx-ingress-controller-but-some-requests-hang-indefinitely-while-others-succeed-what-steps-would-you-take-to-debug-and-isolate-the-cause)

---

## (Optional) Interviewer Prompts

Use these nudges if the candidate stalls:
- How would you distinguish client-side vs server-side stall?  
- What kube-proxy mode considerations or node-local DNS behaviors matter?  
- How do TCP vs HTTP-layer timeouts interplay here?  
- Can connection reuse or keepalive settings contribute?  
- What ingress annotations could create uneven behavior?  
