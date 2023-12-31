

## Angle Selection Algorithm


<img align="right"  height="300"  width = "350"  src="https://github.com/FotiouK/Optimising_Beam_Angles_in_Proton_Therapy_of_Lung_Cancer/assets/108896534/befb2e78-7335-42b0-8ddd-f2a92b72e702">

Welcome to the Angle Selection Algorithm documentation. Our algorithm is designed to assess the impact of incident beam geometry on treatment plan quality in proton therapy of lung cancer. In this comprehensive guide, you'll find detailed information on how our algorithm identifies irradiated voxels for various beam geometries by simulating proton beam paths. We'll delve into the essential components, including transformation matrices for gantry and couch angles, distal edge identification, and the generation of beam rays. Additionally, we'll explore the analysis of Water Equivalent Path Length (WEPL) and Percentage Volume Irradiation(PIV) of Organs-At-Risk (OARs) maps to optimise proton beam geometries. With the integration of the Z-score statistic, we combine these maps to create a unified metric. Whether you're a researcher, medical professional, or simply curious about proton therapy, this documentation will provide insights into our angle selection algorithm's inner workings and applications. So, let's get started with understanding the core of our algorithm.
### Beam Simulation 
To assess the impact of incident beam geometry on plan quality, we developed an angle selection algorithm. This algorithm identifies the irradiated voxels for any given beam geometry by simulating proton beam paths. It relies on couch and gantry angles to determine the relative beam direction, achieved through rotation transformation matrices for an input CT scan oriented along the SI, AP, and RL direction.

**Transformation Matrices:**
<br> For gantry angle (GA) and couch angle (CA), the transformation matrices are defined as follows:

$$ GA = \begin{pmatrix}
    1 & 0 & 0 \\
    0 & \cos(GA) & \sin(GA) \\
    0 & -\sin(GA) & \cos(GA)
\end{pmatrix} \quad
CA = \begin{pmatrix}
    \cos(CA) & 0 & -\sin(CA) \\
    0 & 1 & 0 \\
    \sin(GA) & 0 & \cos(GA)
\end{pmatrix} $$

<br> Instead of directly transforming the scans, we divide the beam path into discrete steps, initiating from the tumour’s distal edge points and inversely simulating the proton beam paths. We utilise a directional vector [0,-1,0] to represent each step's direction, ensuring that for gantry and couch angles of 0 degrees, the beam travels normally towards the patient from the anterior direction. This methodology helps generate the step vector of magnitude 1 describing the beam path for any arbitrary gantry-couch angle combination.



**Distal Edge Identification:**
<br>To identify the tumour’s distal edge for various angle combinations, we employ a binary search approach. The algorithm uses the estimated step distance, derived from beam geometry, and the tumour’s spatial coordinates to search in the opposite direction of the step distance. Starting the search from each voxel within the tumor, the algorithm evaluates adjacent voxels along the beam path. The binary search continues until non-tumor tissue is encountered or a predetermined threshold is reached at the tumor boundary. Successful execution of the binary search results in an array containing the distal edge coordinates of the tumor.

**Generating Beam Rays**
<img align="right" width="300" height="250"  src="../Images/Angle_Selection/p104_Beam_Visualisation.png">
<br> Subsequently, we simulate beam rays inversely, using the estimated distal edge points and the beam step vector. Initiating from the identified distal edge points, the algorithm generates lines representing beam trajectories by incrementing the coordinates in the opposite direction of the beam ray. This iterative process continues until the proton ray reaches the image boundaries, encompassing all distal edge points. To handle steps with decimal places, only the coordinate corresponding to a unique voxel traversed by the beam is rounded up, while the rolling sum of coordinates retains decimal precision. The output consists of a list of lists, where each inner list contains the irradiated voxels along the proton ray's path, terminating at a distal edge point. In the adjacent image, we visualise the simulate beam for gantry angle 45 and couch angle 0 degrees in blue, the tumour in red and the distal edge in yellow. The distal edge was expanded by 5mm for visualisation purposes.


### ΔWEPL and OAR irradiation maps
Optimal proton beam geometry should consider both tumour coverage and minimising dose to organs at risk. We evaluated tumor coverage using Water Equivalent Path Length (WEPL) analysis and assessed organs at risk using Percentage Volume Irradiation (PIV). Calculations were performed for approximately 350 beam orientations, with the 2D maps below depicting  the results for patient 104.

**Tumour Coverage:**
</br> WEPL represents the path a proton beam traverses through water, calculated by summing the relative proton stopping power ratio multiplied by the cohort length. Thus, the RSP converted CT scans from our pre-processing algorithm were used as the input. In the context of proton beam therapy for lung cancer, treatment planning is performed on a static representation of the target volume. A key objective is to minimise the variations in WEPL along the planned and evaluated beam paths, aiming to reduce uncertainties in proton range.
We determined the reference WEPL from planning CT scans (AIP CT from pre-processing) for each proton ray with specific gantry-couch angles. We also computed the WEPLs of each ray for all breathing phases from the converted RSP 4DCT scans. ΔWEPL for each ray was calculated by subtracting the absolute value of reference WEPL from the evaluated WEPL. The average of all proton rays yielded a single-phase proton beam ΔWEPL, while averaging over all phases provided the overall ΔWEPL value representative of the entire beam. 

**Organs At Risk Dose:**
</br> The impact of incident beam geometry on the accumulated dose for organs at risk was evaluated through the Percentage Volume Irradiation (PIV). PIV measures the overlap between the incident beam and the organ, normalised to the total organ volume. The algorithm considered organs at risk such as the heart, lungs, and spinal cord.


<p align="center">
  <img  height="300" src="../Images/Angle_Selection/PIV_WEPL_p104.png">
</p>

### Z-Score Normalisation.
We integrated information from the Water Equivalent Path Length difference (ΔWEPL) and Organ at Risk irradiation maps using the Z-score statistic. This statistical method standardized the values of each map, allowing us to combine them into a single metric. Z-score conversion was applied on every pixel in the ΔWEPL and PIV maps shown above. This transformation turned pixel values into relative variables, indicating their deviation from the population average in terms of standard deviation. Positive Z-scores represented values above the mean, while negative Z-scores indicated values below the mean.
<br> Below, you can see the Z-score-normalised ΔWEPL and PIV maps for patient 104. Upon comparison with the previously mentioned maps, we observed consistent behavioural patterns, with only variations in pixel values.

<p align="center">
  <img height="300" src="../Images/Angle_Selection/Z_Score_Values_p104.png">
</p>

</br>  To create a unified metric, we followed a methodology similar to the treatment planning optimisation process, where weighting factors are applied to plan parameters. In this case, we multiplied each map with distinct weighting factors and then summed them to generate the Final Z-score map. These weighting factors were determined based on various considerations, including patient anatomy, analysis of variable effectiveness maps, and the patient's prior medical history. This approach allowed for adjustments between plan objectives during the angle selection process, enabling a higher flexibility during treatment planning.
<br>  For patient 104, we applied specific weighting factors as follows: a tumour weighting factor of 2, 1.5 for the heart, 1.8 for the lungs, and 0.5 for the spinal cord. The resulting Final Z-Score map is shown below.

<p align="center">
  <img height="300" src="../Images/Angle_Selection/Final_Z_Score_Map_p104.png">
</p>


### Angle Selection 
<img align="left"   src="../Images/Angle_Selection/Central_angle_theorem.png">
</br> Using the Final Z-score map for each patient, we extracted and utilised the three optimal treatment angles for treatment planning. These optimal angles were identified by selecting the three-angle combination with the lowest Z-score. To prevent cross-beam irradiation, we enforced a minimum 20-degree angle separation between all beams. We assessed beam separation based on the Central Angle theorem, which describes the angular separation of two points located on the surface of a sphere. In our case, these two points represented the starting points of the incident beams determined by the couch and gantry angle combination. It's worth noting that the number of beams and beam separation for treatment planning can be adjusted to accommodate patient-specific parameters.

