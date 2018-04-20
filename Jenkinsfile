node('node1') {
    
   // Create a VM to deploy Devstack on it
   stage 'VM Creation'
   
   timeout(time: 60, unit: 'SECONDS') {
       catchError {
           sh '''#!/bin/bash
           # Load virtual env and authenticate to OpenStack
           export WORKON_HOME=$HOME/.virtualenvs
           source /usr/local/bin/virtualenvwrapper.sh
           workon openstack
           source /home/ubuntu/osic-engineering-openrc.sh
         
           echo 'Prepare all the VM variables'
           prefix='ci-devstack-'
           uuid=$(uuidgen)
           name=$prefix$uuid
           image='ubuntu-server-16.04'
           flavor='m1.medium'
           sec_grp='vikas-infra-secgrp'
           keypair='vikas-infra-kp'
           network_id='fd0e26e9-5705-491e-b8ad-4872880c3d77'
         
           echo 'Create the VM, wait for it to finish building'
           openstack server create $name --image $image --flavor $flavor --security-group $sec_grp --key-name $keypair --nic net-id=$network_id --wait
         
           echo 'Get the VM name and status'
           openstack server show $name | grep -w ' name ' | awk '{print $4}' > vm_name
           openstack server show $name | grep status | awk '{print $4}' > vm_status
           '''
       }
   }
   
   def vm_name = readFile('vm_name').trim()
   def vm_status = readFile('vm_status').trim()
   echo "VM name: ${vm_name}, VM status: ${vm_status}"
   
   stage 'Deploy'
   echo 'To be defined...'

}

node('rally_agent') {
    
    stage 'Run Performance Tests'
    
    catchError {
        sh '''#!/bin/bash
        # Load virtual env and authenticate to OpenStack
        export WORKON_HOME=$HOME/.virtualenvs
        source /usr/local/bin/virtualenvwrapper.sh
        workon rally
        source /home/ubuntu/openrc.sh
    
        echo 'Start with a new DB'
        rally-manage db recreate
    
        echo 'Create a new deployment'
        rally deployment create --fromenv --name=vikas-devstack-temp
    
        echo 'Run selected tests'
        rally task start /home/ubuntu/rally/samples/tasks/scenarios/nova/create-flavor.json
        '''
    }
    
    sh '''#!/bin/bash
    # Load virtual env and authenticate to OpenStack
    export WORKON_HOME=$HOME/.virtualenvs
    source /usr/local/bin/virtualenvwrapper.sh
    workon rally
    source /home/ubuntu/openrc.sh
    
    echo 'Saving Rally results to a file'
    rally task report --junit --out results.xml
    '''
    
    def rally_results = readFile('results.xml').trim()
    echo rally_results
    
}

node('node1') {
    
    stage 'Delete VM'
    def vm_name = readFile('vm_name').trim()
    def vm_status = readFile('vm_status').trim()
    println "The VM name is ${vm_name}"
    echo "The status of the VM is ${vm_status}"
    
    timeout(time: 60, unit: 'SECONDS') {
        
        catchError {
            sh '''#!/bin/bash
            # Load virtual env and authenticate to OpenStack
            export WORKON_HOME=$HOME/.virtualenvs
            source /usr/local/bin/virtualenvwrapper.sh
            workon openstack
            source /home/ubuntu/osic-engineering-openrc.sh
       
            # Get the name of the VM
            vm_name=$(head -n 1 vm_name)
       
            # Delete the VM
            echo 'Deleting VM ' $vm_name
            openstack server delete $vm_name
            '''
        }
        
    }
    
}

node('master') {
    
    // Send an email if the build fails
    step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: 'chennybirru@gmail.com', sendToIndividuals: false])

}
