# the_last_test  
```
 __    __                       ___                     __            __                   __      
/\ \__/\ \                     /\_ \                   /\ \__        /\ \__               /\ \__   
\ \ ,_\ \ \___      __         \//\ \      __      ____\ \ ,_\       \ \ ,_\    __    ____\ \ ,_\  
 \ \ \/\ \  _ `\  /'__`\         \ \ \   /'__`\   /',__\ \ \/        \ \ \/  /'__`\ /',__\ \ \/  
  \ \ \_\ \ \ \ \/\  __/          \_\ \_/\ \L\.\_/\__, `\ \ \_        \ \ \_/\  __//\__, `\ \ \_ 
   \ \__\ \_\ \_\ \____\         /\____\ \__/.\_\/\____/ \ \__\        \ \__\ \____\/\____/ \ \__\
    \/__/ \/_/\/_/\/____/  _______\/____/\/__/\/_/\/___/   \/__/  _______\/__/\/____/\/___/   \/__/
                          /\______\                              /\______\                         
                          \/______/                              \/______/                         
```  

Random Linus Torvalds' rant:  
_You messed up the pull request too.. The branch name is missing from that git line, even if you did mention it a few lines earlier..._  

This project was auto generated with the following script:  
```sh
#!/bin/bash

if [ -z "$1" ]
then
    echo "No project name specified"
    exit 1
fi

exec_dir=$(pwd)
project_dir="${HOME}/Projects/$1"

if [ -e "$project_dir" ]
then
    echo "Project with specified name already exists"
    exit 1
fi

# Initialize the project
mkdir -p $project_dir
cd $project_dir
git init

# The title
echo -e "# $1  " > README.md

# The fancy title
fancy_text=$(curl -s https://asciified.thelicato.io/api/v2/ascii\?text=${1}\&font=Larry+3D)
if [ -e "$fancy_text" ]
then
    echo "There was an error when using the API, switching to a generic message"
    fancy_text=$(cat "${exec_dir}/generic_fancy.txt")
fi
echo -e "\`\`\`\n${fancy_text}\n\`\`\`  \n" >> README.md

# Insert a random Linus' rant
rant=$(shuf -n 1 ${exec_dir}/linus_rants.txt)
echo -e "Random Linus Torvalds' rant:  \n_${rant}_  \n" >> README.md

# Insert the code of the currently running script
echo -e "This project was auto generated with the following script:  \n\`\`\`sh" >> README.md
cat $exec_dir/$0 >> README.md
echo -e "\n\`\`\`  \n" >> README.md

nano +-1 README.md

git add README.md
git commit -a -m "Initial commit"

# Generate ssh keys for use with github
ssh-keygen -t ed25519 -N "" -f "${HOME}/.ssh/project_$1" -q
echo -e "Host github-$1.com\n  IdentityFile=${HOME}/.ssh/project_$1\n  Hostname github.com" >> ${HOME}/.ssh/config
key=$(cat ${HOME}/.ssh/project_$1.pub)

# Check if github account info was provided
if [[ -z "$2" || -z "$3" ]]
then
    # Print the public key if info was not provided
    echo -e "\nYour public ssh key:"
    echo -e "$key"
    echo -e "Create a new repository at https://github.com/new"
else
    echo -e "repo.json" > .gitignore
    curl -s -X POST -u "$2:$3" https://api.github.com/user/repos -d "{\"name\":\"linux_lab_$1\"}" > repo.json
    ssh_url=$(jq ".ssh_url" -r repo.json)
    ssh_url=${ssh_url/"github.com"/"github-$1.com"}
    curl -s -u "$2:$3" -X POST https://api.github.com/repos/$2/linux_lab_$1/keys -d "{\"key\":\"${key}\"}"
    git remote add origin $ssh_url
    git push -u origin main
fi

exit 0

```  

Manual addition to the file
