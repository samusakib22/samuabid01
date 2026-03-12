# import cv2
import numpy as np
import subprocess
import os

# -------- STEP 1 : FLOOR PLAN READ --------
image = cv2.imread("plan.png")
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

# Edge detection
edges = cv2.Canny(gray,50,150)

# Contours detect (rooms / walls)
contours,_ = cv2.findContours(edges,cv2.RETR_TREE,cv2.CHAIN_APPROX_SIMPLE)

walls = []
scale = 0.05

for cnt in contours:
    x,y,w,h = cv2.boundingRect(cnt)
    if w > 40 and h > 40:
        walls.append((x*scale,y*scale,w*scale,h*scale))

# -------- STEP 2 : BLENDER SCRIPT --------
blender_script = f"""
import bpy

bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()

def create_wall(x,y,w,h,height=3):
    bpy.ops.mesh.primitive_cube_add(size=1)
    wall=bpy.context.object
    wall.scale=(w/2,h/2,height/2)
    wall.location=(x,y,height/2)

walls={walls}

for w in walls:
    create_wall(*w)

# Floor
bpy.ops.mesh.primitive_plane_add(size=20, location=(0,0,0))

# Kashmiri style roof
bpy.ops.mesh.primitive_cone_add(vertices=4, radius1=10, depth=3, location=(0,0,5))
roof=bpy.context.object
roof.rotation_euler[2]=0.78
"""

# Save blender script
with open("create_house.py","w") as f:
    f.write(blender_script)

# -------- STEP 3 : RUN BLENDER --------
blender_path = "blender"  # agar blender PATH me hai
subprocess.call([blender_path,"--python","create_house.py"])
