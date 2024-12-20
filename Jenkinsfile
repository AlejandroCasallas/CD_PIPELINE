// Pipeline para ejecutar el despliegue en el ambiente QA para el backend de Sable
node {

    // Definición del objeto 'remote' para configurar la conexión SSH
    def remote = [:] // Se crea un mapa vacío para la configuración de la conexión remota
    remote.name = "Ambiente QA" // Nombre descriptivo para la conexión remota, en este caso se refiere al ambiente de QA
    remote.host = "XXX.XXX.XXX.XXX" // Dirección IP del servidor remoto (información sensible eliminada)
    remote.allowAnyHosts = true // Permite la conexión a cualquier host SSH sin verificar la clave

    // Uso de credenciales seguras almacenadas en Jenkins
    // Estas credenciales están guardadas con el ID 'servers-montechelo' y son recuperadas mediante usernamePassword
    // El ID de las credenciales es seguro, evita almacenar el usuario/contraseña directamente en el código
    withCredentials([usernamePassword(credentialsId: 'servers-montechelo', passwordVariable: 'PASS_SERVERS', usernameVariable: 'USER_SERVERS')]) {

        // Asignación del usuario y contraseña al objeto 'remote'
        remote.user = "${USER_SERVERS}" // El usuario se obtiene de las credenciales almacenadas
        remote.password = "${PASS_SERVERS}" // La contraseña también se obtiene de las credenciales almacenadas

        // Definición de la etapa para el despliegue en QA
        stage("Ejecución Despliegue Backs Ambiente QA -> Sable") {

            // Mensaje de inicio para el snapshot
            echo 'Ejecución Snapshot Nutanix'

            // Primer comando SSH: Ejecuta un script para crear un snapshot en el servidor remoto
            sshCommand remote: remote, command: "/var/www/mios/./.deploysnap.sh"

            // Segundo comando SSH: Navega al directorio del backend y actualiza el repositorio Git
            sshCommand remote: remote, command: """
                cd /var/www/mios/mios-backend/lacardio-back && 
                git status && 
                git checkout --force && 
                git pull && 
                php artisan cache:clear
            """
            // Explicación:
            // - 'git status': Verifica el estado actual del repositorio.
            // - 'git checkout --force': Fuerza el cambio a la rama predeterminada (descarta cambios locales).
            // - 'git pull': Actualiza el repositorio con los últimos cambios del servidor remoto.
            // - 'php artisan cache:clear': Limpia la caché de la aplicación para evitar problemas con datos desactualizados.

            // Mensaje de finalización del despliegue RECUERDA SIEMPER CAMBIARLO
            echo 'Despliegue Back QA'
        }
    }
}

// Envío de correo electrónico con los logs adjuntos del proceso de despliegue
// Las variables de contenido y destinatarios están configuradas como predeterminadas en Jenkins
emailext attachLog: true, body: '${DEFAULT_CONTENT}', subject: '${DEFAULT_SUBJECT}', to: '$DEFAULT_RECIPIENTS'
