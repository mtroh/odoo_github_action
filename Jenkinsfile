pipeline {
    agent {
        label 'ubuntu_agent'
    }
    
    environment {
        ODOO_ADDONS_PATH = '/opt/odoo/custom/addons'
        MODULE_NAME = 'mail_debrand_pipeline'
        DB_NAME = 'odoo'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Deploy Module') {
            steps {
                script {
                    sh """
                    # Créer le répertoire des addons si nécessaire
                    if [ ! -d "${ODOO_ADDONS_PATH}" ]; then
                        sudo mkdir -p ${ODOO_ADDONS_PATH}
                    fi
                    
                    # Copier le module
                    sudo cp -r ${MODULE_NAME} ${ODOO_ADDONS_PATH}/
                    
                    # Définir les permissions
                    sudo chown -R odoo:odoo ${ODOO_ADDONS_PATH}/${MODULE_NAME}
                    """
                }
            }
        }
        
        stage('Update & Restart') {
            steps {
                script {
                    sh """
                    # Mettre à jour la liste des modules
                    sudo systemctl stop odoo
                    sudo -u odoo /usr/bin/odoo --config=/etc/odoo/odoo.conf --database=${DB_NAME} --update-module-list --stop-after-init
                    sudo systemctl start odoo
                    
                    # Attendre que le service démarre
                    sleep 5
                    
                    # Vérifier que le module est reconnu
                    echo "Vérification que le module est dans la base de données..."
                    sudo -u odoo psql ${DB_NAME} -c "SELECT name, state FROM ir_module_module WHERE name='${MODULE_NAME}';"
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo 'Module déployé avec succès dans Odoo !'
        }
        failure {
            echo 'Échec du déploiement du module'
        }
    }
}
