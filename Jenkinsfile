pipeline {
    agent {
        label 'ubuntu_agent'  // Utilisez votre agent Ubuntu
    }
    
    environment {
        // Les chemins par défaut puisqu'ils ne sont pas spécifiés dans la config
        ODOO_ADDONS_PATH = '/opt/odoo/custom/addons'
        // Le port xmlrpc est 8069 comme indiqué dans votre config
        ODOO_PORT = '8069'
        // Stocker le mot de passe admin de manière sécurisée
        ADMIN_PASSWORD = credentials('odoo-admin-password')
        // Vérifiez si le répertoire des modules existe, sinon le créer
        MODULE_NAME = 'mail_debrand_pipeline'
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
                    
                    // Obtenir la liste des bases de données Odoo
                    sh """
                    echo "Liste des bases de données PostgreSQL disponibles:"
                    sudo -u postgres psql -c "SELECT datname FROM pg_database WHERE datname NOT IN ('postgres', 'template0', 'template1');"
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
                    // Demander à l'utilisateur de spécifier la base de données si nécessaire
                    def DB_NAME = input(
                        id: 'userInput',
                        message: 'Entrez le nom de la base de données Odoo:',
                        parameters: [
                            string(defaultValue: 'odoo', name: 'DATABASE_NAME', description: 'Nom de la base de données Odoo')
                        ]
                    ).DATABASE_NAME
                    
                    // Mettre à jour la liste des modules via l'API Odoo
                    sh """
                    # Mettre à jour la liste des modules
                    curl -X POST \\
                        -H 'Content-Type: application/json' \\
                        -d '{"jsonrpc": "2.0", "method": "call", "params": {"service": "object", "method": "execute", "args": ["${DB_NAME}", "admin", "${ADMIN_PASSWORD}", "ir.module.module", "update_list"]}}' \\
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
