# k8s-manifest
k8s-manifest/
├── README.md
├── argocd/                           
│   ├── root-app.yaml                
│   └── apps/                          
│       ├── backend-staging.yaml       
│       ├── backend-production.yaml    
│       ├── frontend-staging.yaml      
│       ├── frontend-production.yaml   
│       └── monitoring-stack.yaml      
│
├── apps/                             
│   ├── backend/                       
│   │   ├── Chart.yaml                 
│   │   ├── values-staging.yaml        
│   │   ├── values-production.yaml    
│   │   └── templates/                
│   │       ├── deployment.yaml       
│   │       ├── service.yaml
│   │       └── ingress.yaml
│   │
│   └── frontend/                 
│       ├── Chart.yaml
│       ├── values-staging.yaml
│       ├── values-production.yaml
│       └── templates/
│           ├── deployment.yaml
│           └── service.yaml
│
└── monitoring/                       
    ├── fluentbit/
    │   ├── Chart.yaml                 
    │   └── values.yaml                
    ├── grafana/
    │   └── values.yaml                
    ├── opensearch/
    │   └── values.yaml
    └── prometheus/
        └── values.yaml                