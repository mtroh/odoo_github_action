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
                    # Simplement redémarrer le service Odoo
                    # Odoo scanne les dossiers d'addons au démarrage et met à jour la liste des modules
                    sudo systemctl restart odoo
                    
                    # Attendre que le service démarre complètement
                    echo "Attente du démarrage d'Odoo..."
                    sleep 10
                    
                    # Vérifier si le module est dans la base de données
                    echo "Vérification de la présence du module dans la base de données..."
                    RESULT=\$(sudo -u odoo psql ${DB_NAME} -t -c "SELECT COUNT(*) FROM ir_module_module WHERE name='${MODULE_NAME}';")
                    
                    if [ "\$(echo \$RESULT | tr -d ' ')" -gt "0" ]; then
                        echo "Le module ${MODULE_NAME} a été trouvé dans la base de données!"
                    else
                        echo "Le module ${MODULE_NAME} n'a pas été trouvé dans la base de données."
                        # On va installer manuellement le module en utilisant l'API Python d'Odoo
                        echo "Tentative de mise à jour manuelle de la liste des modules..."
                        
                        sudo -u odoo python3 - <<EOF
import sys, os
try:
    import odoo
    from odoo.modules import module
    odoo.tools.config.parse_config(['-c', '/etc/odoo/odoo.conf', '-d', '${DB_NAME}'])
    with odoo.api.Environment.manage():
        registry = odoo.modules.registry.Registry('${DB_NAME}')
        with registry.cursor() as cr:
            env = odoo.api.Environment(cr, odoo.SUPERUSER_ID, {})
            module_obj = env['ir.module.module']
            print("Mise à jour de la liste des modules...")
            module_obj.update_list()
            print("Liste des modules mise à jour.")
except Exception as e:
    print(f"Erreur lors de la mise à jour: {e}")
    sys.exit(1)
EOF
                        
                        # Vérifier à nouveau
                        echo "Vérification après mise à jour manuelle..."
                        sudo -u odoo psql ${DB_NAME} -c "SELECT name, state FROM ir_module_module WHERE name='${MODULE_NAME}';"
                    fi
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
            echo 'Échec du déploiement du module. Vérifiez les logs pour plus de détails.'
        }
    }
}
