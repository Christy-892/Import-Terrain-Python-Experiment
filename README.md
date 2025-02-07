# Import-Terrain-Python-Experiment
My aim was to create an HDA which could import heightmap files which in turn could be used in other HDAs as a source. I wanted to try making this useful for any production so used python to generate the necessary node network based on the files found in the folder path 
## How To Guide
1. Open HDA in Houdini
2. Ensure files in folder path have suitable names
3. Add folder path to input (must have ` either side of path)
4. Click "Load Terrain"
## Features
### Python Callback Script
Python callback script which generates the node network
```python
import hou
import os

def generate_node_network():
    
    #Set Variables
    node = hou.pwd()
    parent_node = hou.parent()
    folder_path = node.evalParm("folderPath")  
    
    previous_heightfield_file_node = None
    previous_heightfield_patch_node = None
    layer_names = []    
    
    #Allow editing of HDA
    node.allowEditingOfContents(True)
    
    #Loop through each file in folder
    files = os.listdir(folder_path)
    for index, file in enumerate(files):
    
        #Set variables
        file_name = os.path.splitext(file)[0]
        file_path = f"{folder_path}/{file_name}.png"
    
        #Create new node if node does not already exists
        new_node_path = f"/obj/{parent_node}/{node}/{file_name}"
        exisiting_node = hou.node(new_node_path)
        if exisiting_node is not None:
           print(f"Node '{exisiting_node}' already exists")
        else:
            #First iteration
            if index == 0:
                
                #Create node
                heightfield_file_node = node.createNode("heightfield_file", file_name)
                previous_heightfield_file_node = heightfield_file_node
                
                #Set parameters
                heightfield_file_node.parm("filename").set(file_path)
                heightfield_file_node.parm("heightscale").set(400)
                heightfield_file_node.parm("size").set(1525)               
                
                #Connect to existing nodes
                hou.node("Isolated").setInput(index, heightfield_file_node)
                hou.node("Merged").setInput(index, heightfield_file_node)
                
                #Append name to layer list
                layer_names.append(f"{index}={file_name}")
                        
                continue
            
            #Create nodes
            heightfield_file_node = node.createNode("heightfield_file", file_name)
            heightfield_mask_node = node.createNode("heightfield_maskbyfeature", file_name)
            heightfield_patch_node = node.createNode("heightfield_patch", file_name)
            
            ##Set parameters##
            #File node
            heightfield_file_node.parm("filename").set(file_path)
            heightfield_file_node.parm("heightscale").set(400)
            heightfield_file_node.parm("size").set(1525)
            
            #Mask node
            heightfield_mask_node.parm("invertmask").set(1)
            heightfield_mask_node.parm("min_slopeangle").set(0)
            heightfield_mask_node.parm("max_slopeangle").set(0)
            
            #Patch node
            heightfield_patch_node.parm("centerpatch").set(0)
            
            ##Connect new nodes##
            if index == 1:
                heightfield_patch_node.setInput(0, previous_heightfield_file_node)
            else:
                heightfield_patch_node.setInput(0, previous_heightfield_patch_node)
            heightfield_mask_node.setInput(0, heightfield_file_node)    
            heightfield_patch_node.setInput(1, heightfield_mask_node)
            
            #Connect to existing nodes
            hou.node("Isolated").setInput(index, heightfield_file_node)
            hou.node("Merged").setInput(index, heightfield_patch_node)
        
            #update
            previous_heightfield_file_node = heightfield_file_node
            previous_heightfield_patch_node = heightfield_patch_node

            layer_names.append(f"{index}={file_name}")              

    #Layout nodes for clarity
    node.layoutChildren()
    
    #Display Info on HDA
    # Get the parameter template group from the node
    parm_template_group = node.parmTemplateGroup()
    
    # Create a label parameter template with multiple column labels
    label_parm_template = hou.LabelParmTemplate("multi_label", "Layers:",layer_names)
    
    # Add the label parameter template to the parameter template group
    parm_template_group.addParmTemplate(label_parm_template)
    
    # Apply the parameter template group back to the node
    node.setParmTemplateGroup(parm_template_group)
```
## Setup
1. Download otl
2. Ensure heightmap files are all in same folder
## Further Reading
- SideFX main Houdini Python Documentation - https://www.sidefx.com/docs/houdini/hom/index.html
- Sub-modules, classes and functions accessable in Houdini - https://www.sidefx.com/docs/houdini/hom/hou/index.html
## ToDo
- [x] Proof of Concept
