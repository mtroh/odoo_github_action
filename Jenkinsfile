pipeline {
    agent {
        label 'ubuntu_agent'  // Utilise votre agent Ubuntu
    }
    
    environment {
        // Chemins et configurations
        ODOO_ADDONS_PATH = '/opt/odoo/custom/addons'
        ODOO_PORT = '8069'
        MODULE_NAME = 'mail_debrand_pipeline'
        DB_NAME = 'odoo'  // Nous savons maintenant que la base de données est 'odoo'
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Récupération du code depuis GitHub
                checkout scm
            }
        }
        
        stage('Prepare Environment') {
            steps {
                script {
                    // Vérifier si le répertoire des addons existe, sinon le créer
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
                    // Déployer le module
                    sh """
                    # Copier le module dans le répertoire des addons
                    sudo cp -r ${MODULE_NAME} ${ODOO_ADDONS_PATH}/
                    
                    # Définir les bonnes permissions
                    sudo chown -R odoo:odoo ${ODOO_ADDONS_PATH}/${MODULE_NAME}
                    """
                }
            }
        }
        
        stage('Update Module List') {
            steps {
                script {
                    // Comme la DB est en trust mode, nous pouvons nous connecter sans mot de passe
                    // En utilisant 'admin' comme utilisateur Odoo par défaut
                    sh """
                    # Mettre à jour la liste des modules
                    curl -X POST \\
                        -H 'Content-Type: application/json' \\
                        -d '{"jsonrpc": "2.0", "method": "call", "params": {"service": "object", "method": "execute", "args": ["${DB_NAME}", "admin", "admin", "ir.module.module", "update_list"]}}' \\
                        http://localhost:${ODOO_PORT}/jsonrpc
                    
                    # Redémarrer le service Odoo pour appliquer les changements
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
