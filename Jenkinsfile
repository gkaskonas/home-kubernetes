pipeline {
    agent any
    environment {
        // The MY_KUBECONFIG environment variable will be assigned
        // the value of a temporary file.  For example:
        //   /home/user/.jenkins/workspace/cred_test@tmp/secretFiles/546a5cf3-9b56-4165-a0fd-19e2afe6b31f/kubeconfig.txt
        KUBECONFIG = credentials('kubeconfig_minikube')
    }    
    stages {
        stage('Set up Helm') {
            steps {
                bat "helm repo add stable https://kubernetes-charts.storage.googleapis.com"
                bat "helm repo add jetstack https://charts.jetstack.io"
                bat "helm repo update"
            }
        }
        stage('Install MetalLB') {
            steps {
                bat "helm upgrade metallb stable/metallb --namespace kube-system \
                      --set configInline.address-pools[0].name=default \
                      --set configInline.address-pools[0].protocol=layer2 \
                      --set configInline.address-pools[0].addresses[0]=192.168.0.240-192.168.0.250 \
                      --install"
            }
        }
        stage('Install Nginx Ingress') {
            steps {
                bat "helm upgrade nginx-ingress stable/nginx-ingress --namespace kube-system \
                        --set controller.image.repository=quay.io/kubernetes-ingress-controller/nginx-ingress-controller \
                        --set controller.image.tag=0.30.0 \
                        --set defaultBackend.enabled=false \
                        --install"
            }
        }
        stage('Install Cert Manager') {
            steps {
                bat "kubectl --kubeconfig $KUBECONFIG apply -f ./cert-manager/namespace.yaml"
                bat "kubectl --kubeconfig $KUBECONFIG apply -f ./cert-manager/00-crds.yaml"
                bat "helm upgrade cert-manager jetstack/cert-manager --namespace cert-manager --install -f ./helm_values/certmanager_values.yaml"
                bat "kubectl --kubeconfig $KUBECONFIG apply -f ./cert-manager/cluster-issuer-prod.yaml"
                bat "kubectl --kubeconfig $KUBECONFIG apply -f ./cert-manager/cluster-issuer-staging.yaml"
            }
        }
        stage('Install Rocketchat') {
            steps {
                bat "kubectl --kubeconfig $KUBECONFIG apply -f ./rocketchat/namespace.yaml"
                bat 'helm upgrade --install rocketchat stable/rocketchat --namespace rocketchat -f ./helm_values/rocketchat_values.yaml'
                bat "kubectl --kubeconfig $KUBECONFIG apply -f ./rocketchat/ingress.yaml"
            }
        }
        stage('Install Nexus') {
            steps {
                bat "kubectl --kubeconfig $KUBECONFIG apply -f ./nexus"
            }
        }
        stage('Install NextCloud') {
            steps {
                bat "kubectl --kubeconfig $KUBECONFIG apply -f ./nextcloud/namespace.yaml"
                bat "kubectl --kubeconfig $KUBECONFIG apply -f ./nextcloud/persistentvolume.yaml"
                bat "kubectl --kubeconfig $KUBECONFIG apply -f ./nextcloud/persistentvolumeclaim.yaml"
                bat 'helm upgrade --install nextcloud stable/nextcloud --namespace nextcloud -f ./helm_values/nextcloud_values.yaml'
                bat "kubectl --kubeconfig $KUBECONFIG apply -f ./nextcloud/ingress.yaml"
            }
        }
        stage('Install Kubernetes Dashboard') {
            steps {
                bat "kubectl --kubeconfig $KUBECONFIG apply -f ./kubernetes-dashboard/all.yaml"
            }
        }
    }
}
