# 4Band_Raster
This function gets a hyperspectral raster, searches for four specific band wavelengths and outputs a raster of those four spectral bands with the same file format as the input. The default spectral wavelengths is [440, 540, 640, 840] nm which corresponds to the Blue, Green, Red and Near Infrared parts of spectrum. 
# shahbazi.anahitaa@gmail.com

import rasterio as rio
import os
import numpy as np
##############################################################################
# Base directory
base_dir = r"U:\users\Anahita_Shahbazi\Python_Coding_Practice\01_4band_raster\dataset"
default_wavelength = [440, 540, 640, 840]
##############################################################################
# Read the header file of one of flightlines in pix_files folder to extract channels wavelengths
pix_hdr_dir = os.path.join(base_dir,"pix_files")
for i in os.listdir(pix_hdr_dir):
    name, hdr = os.path.split(i)
    if hdr[-8:] == '.pix.hdr':
        wavelength = []
        hdr_filepath = os.path.join(pix_hdr_dir,hdr)
        with open(hdr_filepath) as header:
            for line in header:
                if line[0:7] == 'Channel':
                    channel = line.split()
                    temp = channel[2]
                    wavelength.append(float(temp.replace('nm','')))
        break
# Look for and choose the band number and wavelength by comparing them with the default wavelengths for 4-band raster
wavelength = np.array(wavelength)
band_idx = []
applied_wl = []
for i in range(len(default_wavelength)):
    dwl = abs(wavelength - default_wavelength[i])
    min_dwl = min(dwl)
    for j in range(len(dwl)):
       if dwl[j] == min_dwl:
           band_idx.append(j)
           applied_wl.append(wavelength[j])
           break
band_idx = np.array(band_idx)
band_idx = band_idx + 1
band_idx = band_idx.tolist()
applied_wl = np.array(applied_wl)
applied_wl = applied_wl.tolist()
print("Applied channels for the 4-band raster are: ",band_idx)
print("The channels correspond to these wavelengths:",applied_wl)
##############################################################################
# Create 4band_tiles sub-folder if it doesn't exist
out_dir = os.path.join(base_dir,"mosaics","4band_tiles")
isExist = os.path.exists(out_dir)
if not isExist:
    os.makedirs(out_dir)
##############################################################################
# Start reading all the hyperspectral imagery tiles and write 4band image for each tile
mosaic_dir = os.path.join(base_dir,"mosaics")
for i in os.listdir(mosaic_dir):
    name, ext = os.path.splitext(i)
    if ext == '.bsq':
        # Open the input hyperspectral image
        img_filepath = os.path.join(mosaic_dir, name) + ext
        img_file = rio.open(img_filepath)
        # Info about the location and resolution of the image, coordinate system and number of the bands
        profile = img_file.profile
        # Update number of the bands for the output 4-band image
        profile.update(count=4)
        # write 4 band image
        output_img = os.path.join(out_dir, name) + "_4band" + ext
        with rio.open(output_img,
                      mode = 'w',
                      **profile) as new_dataset:
            new_dataset.write(img_file.read(band_idx))
