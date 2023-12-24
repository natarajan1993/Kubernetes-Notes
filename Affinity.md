- Affinity is a group of affinity scheduling rules 
`kubectl explain pod.spec.affinity`
```md
- nodeAffinity  <NodeAffinity>  
   Describes node affinity scheduling rules for the pod.  
  
- podAffinity   <PodAffinity>  
   Describes pod affinity scheduling rules (e.g. co-locate this pod in the same node, zone, etc. as some other pod(s)).  
  
- podAntiAffinity       <PodAntiAffinity>  
   Describes pod anti-affinity scheduling rules (e.g. avoid putting this pod in  
   the same node, zone, etc. as some other pod(s)).
```
