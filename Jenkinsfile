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
            sh """
            # Mettre à jour la liste des modules avec le mot de passe administrateur
            curl -s -X POST \\
                -H 'Content-Type: application/json' \\
                -d '{"jsonrpc": "2.0", "method": "call", "params": {"master_password": "va8r-kxnp-t538", "database": "odoo", "action": "modules"}}' \\
                http://localhost:8069/web/database/manager
            
            # Redémarrer le service Odoo
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
