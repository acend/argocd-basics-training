---
title: "12. Private Git repositories"
weight: 12
sectionnumber: 12
onlyWhen: implemented
---

## Examples

TODO

```
Examples:                                                                                                                                                     
  # Add a Git repository via SSH using a private key for authentication, ignoring the server's host key:                                                      
  argocd repo add git@git.example.com:repos/repo --insecure-ignore-host-key --ssh-private-key-path ~/id_rsa                                                   
                                                                                                                                                              
  # Add a Git repository via SSH on a non-default port - need to use ssh:// style URLs here                                                                   
  argocd repo add ssh://git@git.example.com:2222/repos/repo --ssh-private-key-path ~/id_rsa                                                                   
                                                                                                                                                              
  # Add a private Git repository via HTTPS using username/password and TLS client certificates:                                                               
  argocd repo add https://git.example.com/repos/repo --username git --password secret --tls-client-cert-path ~/mycert.crt --tls-client-cert-key-path ~/mycert.
key                                                                                                                                                           
                                                                                                                                                              
  # Add a private Git repository via HTTPS using username/password without verifying the server's TLS certificate                                             
  argocd repo add https://git.example.com/repos/repo --username git --password secret --insecure-skip-server-verification                                     
                                                                                                                                                              
  # Add a public Helm repository named 'stable' via HTTPS                                                                                                     
  argocd repo add https://charts.helm.sh/stable --type helm --name stable                                                                                     
                                                                                                                                                              
  # Add a private Helm repository named 'stable' via HTTPS                                                                                                    
  argocd repo add https://charts.helm.sh/stable --type helm --name stable --username test --password test                                                     
                                                                                                                                                              
  # Add a private Helm OCI-based repository named 'stable' via HTTPS                                                                                          
  argocd repo add helm-oci-registry.cn-zhangjiakou.cr.aliyuncs.com --type helm --name stable --enable-oci --username test --password test                     
                                                                                                                                                              
  # Add a private Git repository on GitHub.com via GitHub App                                                                                                 
  argocd repo add https://git.example.com/repos/repo --github-app-id 1 --github-app-installation-id 2 --github-app-private-key-path test.private-key.pem      
                                                                                                                                                              
  # Add a private Git repository on GitHub Enterprise via GitHub App                                                                                          
  argocd repo add https://ghe.example.com/repos/repo --github-app-id 1 --github-app-installation-id 2 --github-app-private-key-path test.private-key.pem --git
hub-app-enterprise-base-url https://ghe.example.com/api/v3                                                                                        
```