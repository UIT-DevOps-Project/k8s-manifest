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
│       ├── monitoring.yaml            
│       ├── fluentbit.yaml             
│       └── opensearch.yaml            
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
    │   ├── Chart.lock                 
    │   ├── Chart.yaml                 
    │   ├── charts/
    │   └── values.yaml                
    ├── opensearch/
    │   ├── Chart.lock
    │   ├── Chart.yaml
    │   ├── charts/
    │   └── values.yaml
    └── prometheus/
        ├── Chart.lock
        ├── Chart.yaml
        ├── charts/
        └── values.yaml                