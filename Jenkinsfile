// ====== IPs privadas (tu arquitectura) ======
def ansible_server_private_ip = "172.31.78.95"     // Ansible Controller
def kubernetes_server_private_ip = "172.31.64.9"   // Webnode (k3s)
def nodeport = "30000"

// ====== IMPORTANTE: pon aquí el DNS o IP PÚBLICA del webnode para el curl externo ======
def webnode_public = "ec2-3-235-243-197.compute-1.amazonaws.com"

node {

  stage('Connectivity: Jenkins -> Ansible Controller') {
    sshagent(['ansible-server']) {
      sh """
        ssh -o StrictHostKeyChecking=no ubuntu@${ansible_server_private_ip} '
          echo OK_CONNECTED_TO_ANSIBLE
          hostname
          whoami
        '
      """
    }
  }

  stage('Connectivity: Ansible -> Webnode (Ansible ping)') {
    sshagent(['ansible-server']) {
      sh """
        ssh -o StrictHostKeyChecking=no ubuntu@${ansible_server_private_ip} '
          ansible -i /etc/ansible/hosts -m ping webnode
        '
      """
    }
  }

  stage('Run Playbook 01-check-webnode.yml') {
    sshagent(['ansible-server']) {
      sh """
        ssh -o StrictHostKeyChecking=no ubuntu@${ansible_server_private_ip} '
          ansible-playbook -i /etc/ansible/hosts /home/ubuntu/ansible/playbooks/01-check-webnode.yml
        '
      """
    }
  }

  stage('Run Playbook 02-deploy-demo.yml (nginx)') {
    sshagent(['ansible-server']) {
      sh """
        ssh -o StrictHostKeyChecking=no ubuntu@${ansible_server_private_ip} '
          ansible-playbook -i /etc/ansible/hosts /home/ubuntu/ansible/playbooks/02-deploy-demo.yml
        '
      """
    }
  }

  stage('Validate k3s: pods + service NodePort') {
  sshagent(['ansible-server']) {
    sh """
      ssh -o StrictHostKeyChecking=no ubuntu@${ansible_server_private_ip} '
        echo "---- kubectl get nodes ----"
        ansible -i /etc/ansible/hosts webnode -b -m shell -a "KUBECONFIG=/etc/rancher/k3s/k3s.yaml kubectl get nodes"

        echo "---- kubectl get pods -A ----"
        ansible -i /etc/ansible/hosts webnode -b -m shell -a "KUBECONFIG=/etc/rancher/k3s/k3s.yaml kubectl get pods -A"

        echo "---- kubectl get svc -A ----"
        ansible -i /etc/ansible/hosts webnode -b -m shell -a "KUBECONFIG=/etc/rancher/k3s/k3s.yaml kubectl get svc -A"

        echo "---- demo-svc must expose NodePort 30000 ----"
        ansible -i /etc/ansible/hosts webnode -b -m shell -a "KUBECONFIG=/etc/rancher/k3s/k3s.yaml kubectl get svc demo-svc -o wide | grep 30000"
      '
    """
  }

  }

  stage('External HTTP check (NodePort 30000)') {
    sh """
      echo "Hitting http://${webnode_public}:${nodeport}"
      curl -f --max-time 10 http://${webnode_public}:${nodeport} | head -n 20
    """
  }
}
