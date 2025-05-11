pipeline {
    agent {
        label 'ubuntu_agent'
    }
    
    environment {
        ODOO_ADDONS_PATH = '/opt/odoo/custom/addons'
        ODOO_PORT = '8069'
        MODULE_NAME = 'mail_debrand_pipeline'
        DB_NAME = 'odoo'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Prepare Environment') {
            steps {
                script {
                    sh """
                    if [ ! -d "${ODOO_ADDONS_PATH}" ]; then
                        sudo mkdir -p ${ODOO_ADDONS_PATH}
                    fi
                    """
                }
            }
        }
        
        stage('Deploy to Odoo') {
            steps {
                script {
                    sh """
                    sudo cp -r ${MODULE_NAME} ${ODOO_ADDONS_PATH}/
                    sudo chown -R odoo:odoo ${ODOO_ADDONS_PATH}/${MODULE_NAME}
                    """
                }
            }
        }
        
        stage('Update Module List') {
            steps {
                script {
                    // Corrigé : Utiliser 1 comme ID utilisateur (l'administrateur a généralement l'ID 1)
                    sh """
                    curl -X POST \\
                        -H 'Content-Type: application/json' \\
                        -d '{"jsonrpc": "2.0", "method": "call", "params": {"service": "object", "method": "execute", "args": ["${DB_NAME}", 1, "admin", "ir.module.module", "update_list"]}}' \\
                        http://localhost:${ODOO_PORT}/jsonrpc
                    
                    sudo systemctl restart odoo
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo 'Module déployé avec succès!'
        }
        failure {
            echo 'Échec du déploiement du module'
        }
    }
}
