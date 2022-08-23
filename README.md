# FF-ref-Calculation-IS-17550
FF Refrigerator IS 17550 Calculation 






New Norms Energy calculation Sheet ( 17550)

Import all essential library
In [161]:
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import r2_score
import matplotlib.pyplot as plt
import seaborn as sns
import warnings
warnings.filterwarnings("ignore")

import io

Import Raw for calculation
In [162]:
VT_df = pd.read_excel(r'D:\HAIL_DATA-1\E drive Data\2022\New Energy Norms _2023\VT8_GG12\R04L2-VT8_G1_CC.xlsx' , sheet_name='Data')
In [163]:

# Display coloum of raw data C.

VT_df.columns
Out[163]:
Index(['Time', 'Date Time', 'Flag', 'Ambient Temp.', 'Ambient Humidity',
       'Suction Pressure', 'Discharge Pressure', 'Voltage', 'Current', 'Power',
       'Int. Power', 'Int_Power', 'Frequency', 'Power Factor', 'Integ. Time',
       'Run. Time Ratio', 'Running Count', 'EVA IN', 'EVA OUT', 'F1', 'F2',
       'F1/2', 'F4', 'F5', 'Favg', 'TFR ', 'R1/3', 'R2/3', 'R25MM', 'Ravg',
       'VEG BOX', 'SUCTION', 'COMPRESSOR', 'HOT LINE', 'DRIER', 'DISCHARGE',
       'AMB-01', 'TC_19', 'AMB-02'],
      dtype='object')
In [164]:

Power = VT_df['Power']
Power =np.array(Power)
print('Length of data point in raw data: ', Power.shape)

plt.figure(figsize=(500, 100))
plt_1 = plt.figure(figsize=(6, 3))

plt.title("Power Graph ")
plt.xlabel("Time ")
plt.ylabel("Power (W) ")
plt.plot(Power, color = 'r', linewidth = '10')
plt.show()

Length of data point in raw data:  (11600,)

 
In [165]:
# Define all require value to futher calculate the parameter

Freezer_Temp = VT_df['Favg']
Fridge_Temp = VT_df['Ravg']
Int_Energy = VT_df ['Int_Power']

# Defrost calculation
In [166]:
Defrost_start = []
x=len(Power) 
for c in range (x-1):
    if (Power[c+1]-Power[c]) > 100:
        defrost = float(c)
        Defrost_start.append(defrost)

print(Defrost_start)

Defrost_end = []
for c in range (x-1):
    if (Power[c]-Power[c+1]) > 100:
        defrost = float(c)
        Defrost_end.append(defrost)

print(Defrost_end)

print('No of defrost in data :', len(Defrost_end))

[1708.0, 11414.0]
[1748.0, 11454.0]
No of defrost in data : 2

# Data cleaning for on off cycle because too much flutuation in data
In [167]:
def cycle_time_data_clean (xxx):

    xxx = np.array (xxx)
    YY = []
    for i in range (len(xxx)):
        if xxx[i] > 60:
            YY.append(i)   
    xxx = np.delete(xxx, YY)
    
    YY = []
    for j in range (len(xxx)):
        if xxx[j] < 35 and xxx[j] >10:
            YY.append(j)   
    xxx = np.delete(xxx, YY)    
    
    
    return xxx

## All paramter define for X period
In [168]:

# data for power in X period
aa = int(Defrost_start[0])
Power_x = Power[:aa,]

Power_x_period = Power_x
Power_x  =  cycle_time_data_clean (Power_x)
print('No of data length in X period ', len(Power_x))

No of data length in X period  1708
In [169]:

## Plot X period 

plt.figure(figsize=(500, 100))
plt.plot(Power_x, color = 'r', linewidth = '10')
Out[169]:
[<matplotlib.lines.Line2D at 0x1bcf2b25190>]

 

# Average Power calculation in Each Period ( function )
In [170]:

def average_power (raw_data):
    f = len(raw_data)
    number_cycle =[]
    for e in range (f-1):
        if (raw_data[e+1]-raw_data[e]) > 20:
            cycle = int(e)
            number_cycle.append(cycle)
                  
    cycle_avg = []
    no_cycle =len(number_cycle)
    for t in range (no_cycle-1):
        a = int (number_cycle[t])
        b = int (number_cycle[t+1])
        cycle_a = np.mean (raw_data[a:b])
        cycle_avg.append(cycle_a)
    return cycle_avg ,number_cycle

# On Off Cycle calculation based on function
In [171]:

def on_off_cycle (raw_data):
    q = len(raw_data)
    on_cycle =[]        
    for e in range (q-1):
        if (raw_data[e+1]-raw_data[e]) > 20:
            On_cycle = int(e)
            on_cycle.append(On_cycle)

    q = len(raw_data)
    off_cycle =[]        
    for e in range (q-1):
        if (raw_data[e]-raw_data[e+1]) > 20:
            Of_cycle = int(e)
            off_cycle.append(Of_cycle)
            
        
    return on_cycle , off_cycle

# Compressor Run percentage in each period ( function)
In [172]:

def run_per (on , off):
    percentage = []
    x = len (on)
    for i in range (x-2):
        n = on[i+1] - on[i]
        f = on[i] - off[i]
        per = 1- (f / n)
        percentage.append(per)
    return percentage

# Cycle average value in X period calculation
In [173]:

cycle_average_value, no_cycle_x_period = average_power (Power_x)
length = len (cycle_average_value)
print('Number of cycle in X period: ', length )
print('Each cycle avergae: ', cycle_average_value)

Number of cycle in X period:  18
Each cycle avergae:  [27.213207456863152, 0.800000011920929, 26.98962257774371, 26.54953276609706, 26.99065423290306, 27.26320759586568, 0.800000011920929, 26.94112140163083, 26.89813092490223, 27.106481494175064, 26.943925204678116, 26.98785054850801, 26.848598200027073, 26.98598129894132, 27.034259364008904, 26.75092591566068, 38.454053973829424, 0.800000011920929]
In [174]:
print ('Cycle value start and end point: ',  no_cycle_x_period )

Cycle value start and end point:  [66, 172, 173, 279, 386, 493, 599, 600, 707, 814, 922, 1029, 1136, 1243, 1350, 1458, 1566, 1640, 1641]

# On off cycle in x period and compressor Run
In [175]:
Power_x_period  = cycle_time_data_clean (Power_x_period)
X_on_cycle , X_off_cycle = on_off_cycle (Power_x_period)

print ('On Cycle value start point: ',  X_on_cycle )
print ('Off Cycle value start  point: ',  X_off_cycle )

On Cycle value start point:  [66, 172, 278, 385, 492, 598, 705, 812, 920, 1027, 1134, 1241, 1348, 1456, 1564, 1638]
Off Cycle value start  point:  [19, 126, 231, 337, 445, 552, 657, 765, 873, 980, 1087, 1194, 1301, 1409, 1516, 1624, 1695]
In [176]:

# Comprssor run percentage in x period
comp_run_X = run_per (X_on_cycle , X_off_cycle) 
print ('Cycle wise run percentage ', comp_run_X )

Cycle wise run percentage  [0.5566037735849056, 0.5660377358490566, 0.5607476635514019, 0.5514018691588785, 0.5566037735849056, 0.5700934579439252, 0.5514018691588785, 0.5648148148148149, 0.5607476635514019, 0.5607476635514019, 0.5607476635514019, 0.5607476635514019, 0.5648148148148149, 0.5648148148148149]

# Property calculation for each cycle points
In [177]:
 def all_temp_data (no_cycle_x_period, req_data):
    data_store= []
    ln = len (no_cycle_x_period)
    for i in range (ln-1):
        a = no_cycle_x_period[i]
        b = no_cycle_x_period[i+1]
        xx = np.mean(req_data[a:b])
        data_store.append(xx)             
    return data_store
    

# All value in X period, Avg Power, Frezeer Temp, Fridge temp
In [178]:

Power_xxx = all_temp_data (no_cycle_x_period, Power_x)
print(Power_xxx)

[27.213207456863152, 0.800000011920929, 26.98962257774371, 26.54953276609706, 26.99065423290306, 27.26320759586568, 0.800000011920929, 26.94112140163083, 26.89813092490223, 27.106481494175064, 26.943925204678116, 26.98785054850801, 26.848598200027073, 26.98598129894132, 27.034259364008904, 26.75092591566068, 38.454053973829424, 0.800000011920929]
In [179]:
Power_xxx = all_temp_data (no_cycle_x_period, Power_x)
print('Stable each cycle data ', Power_xxx)

power_avg_x =np.mean(Power_xxx[(length-8):(length-2)])
print('Average Power in X Period :', power_avg_x)

Stable each cycle data  [27.213207456863152, 0.800000011920929, 26.98962257774371, 26.54953276609706, 26.99065423290306, 27.26320759586568, 0.800000011920929, 26.94112140163083, 26.89813092490223, 27.106481494175064, 26.943925204678116, 26.98785054850801, 26.848598200027073, 26.98598129894132, 27.034259364008904, 26.75092591566068, 38.454053973829424, 0.800000011920929]
Average Power in X Period : 26.925256755304016
In [180]:

# Freezer Room average temperature in X period 
freezer_temp_x = all_temp_data (no_cycle_x_period, Freezer_Temp)
print('Stable each cycle data frezzer room ', freezer_temp_x)

freezer_avg_x =np.mean(freezer_temp_x[(length-8):(length-2)])
print('Average Freezer in X Period :', freezer_avg_x)
#print(Power_xxx)
#Freezer_avg_x =np.mean(no_cycle_x_period[(length-8):])
#print('Average Freezer Temperature  X Period :', Freezer_avg_x)

Stable each cycle data frezzer room  [-18.717773567055758, -17.822000122070314, -18.736811319387183, -18.74973828502905, -18.690224287443066, -18.66222643222449, -17.793999862670898, -18.7035514261121, -18.671794397585856, -18.65088888274299, -18.68439254760742, -18.67611216964009, -18.705457911981604, -18.672654212523845, -18.674777760329068, -18.689092568997975, -18.42464862256437, -20.147999954223632]
Average Freezer in X Period : -18.683747861846665
In [181]:

# Frride Room average temperature in X period 
fridge_temp_x = all_temp_data (no_cycle_x_period, Fridge_Temp)
print('Stable each cycle data fridge room ', fridge_temp_x)

fridge_avg_x =np.mean(fridge_temp_x[(length-8):(length-2)])
print('Average Fridge in X Period :', fridge_avg_x)

Stable each cycle data fridge room  [3.423584912558022, 4.456666628519694, 3.424465401742444, 3.4197819249652266, 3.4330841187747483, 3.4527673005307995, 4.493333339691162, 3.450716508883182, 3.4829906503730843, 3.5081172809924612, 3.4799065474780546, 3.476604360100637, 3.458660444923648, 3.452959497026936, 3.481172828578654, 3.4704629644567566, 3.285720715383153, 3.189999977747599]
Average Fridge in X Period : 3.4699611070941145
In [182]:
x_time_End = no_cycle_x_period[-1]
print ('No of cycle period last time = ', no_cycle_x_period[-1]) 
x_time_Start = no_cycle_x_period[-8]
print ('No of cycle period first time= ', no_cycle_x_period[-8]) 
x_Time = (x_time_End - x_time_Start ) /120 
print ('X Period in Hours  = ', x_Time      )

No of cycle period last time =  1641
No of cycle period first time=  1243
X Period in Hours  =  3.316666666666667

# Calculate all value in defrost to defrost cycle
In [183]:

# data for power in Y period
ab = int(Defrost_end[0])
bb = int(Defrost_start[1])
Power_y = Power[ab:bb,]

print(Power_y)
print ('Y period data length: ',Power_y.shape )

[152.1000061    0.80000001   0.80000001 ...   1.70000005   1.70000005
   1.60000002]
Y period data length:  (9666,)
In [184]:

## Plot Between defrost to defrost period 

plt.figure(figsize=(500, 100))
plt.plot(Power_y, color = 'r', linewidth = '10')
#plt.title("Value between two derost")
Out[184]:
[<matplotlib.lines.Line2D at 0x1bcf26c3970>]

 
In [185]:
cycle_average_value_y, no_cycle_y_period = average_power (Power_y)
length = len (cycle_average_value_y)
print('Number of cycle in Defrost to Defrost period: ', length )
print('Each cycle avergae: ', cycle_average_value_y)

Number of cycle in Defrost to Defrost period:  98
Each cycle avergae:  [39.429629604757565, 29.425892937396252, 27.902803723500153, 27.55047614177068, 27.080000027588436, 27.342307808307503, 27.295192254277374, 27.16952376252129, 26.97333328099478, 27.083809601409094, 27.340384670175037, 27.324761923154195, 1.5, 27.10666675908225, 27.063809573082697, 27.050476187183744, 27.21509427057122, 27.128571450710297, 27.413084031265473, 27.072380935010457, 27.12000009786515, 27.048113069444334, 27.311320766525448, 27.14666662954149, 0.800000011920929, 27.350476199672336, 26.82710286668528, 26.74716967288053, 26.996190487486974, 27.055238097054616, 26.897142826375507, 26.752381009147282, 27.215238137472245, 27.119811252040684, 26.94190479687282, 0.800000011920929, 27.27523808649608, 27.19716961541266, 27.502803640944936, 27.00095253898984, 27.123584868210667, 27.01214965982972, 27.252830236025577, 26.808411191, 26.980373808156664, 27.24433951332884, 27.413084281382158, 27.217924683161502, 26.773831717321805, 26.797169960332365, 27.2660377925297, 27.261320757978368, 27.081308434499757, 27.133962269661563, 27.266981217096436, 0.800000011920929, 27.08679234756614, 26.880373701313946, 27.189622653542823, 0.800000011920929, 26.98396229069188, 26.910280448811076, 27.149056575770647, 26.956074756996653, 27.024299183181512, 27.222429759591538, 27.20754723054058, 27.00747669077365, 26.990654090297557, 26.992523326494982, 26.991588737920065, 26.891509378293776, 26.87943919406873, 26.95233644001952, 26.881308464246377, 26.899065331320898, 26.937383119190965, 26.98317769587597, 26.98411220200708, 26.914018709525884, 26.91869155714445, 27.21775699831615, 27.218518593245083, 27.292523371282027, 0.800000011920929, 27.12336444186273, 27.068518630332417, 27.088784968741585, 27.143518302727628, 26.894392555005084, 27.087850380166667, 27.17169816201588, 0.8999999761581421, 27.023584826937263, 0.800000011920929, 27.1813084148915, 26.66915893666098, 38.78133322238922]
In [186]:
Power_yyy = all_temp_data (no_cycle_y_period, Power_y)
print(Power_yyy)

[39.429629604757565, 29.425892937396252, 27.902803723500153, 27.55047614177068, 27.080000027588436, 27.342307808307503, 27.295192254277374, 27.16952376252129, 26.97333328099478, 27.083809601409094, 27.340384670175037, 27.324761923154195, 1.5, 27.10666675908225, 27.063809573082697, 27.050476187183744, 27.21509427057122, 27.128571450710297, 27.413084031265473, 27.072380935010457, 27.12000009786515, 27.048113069444334, 27.311320766525448, 27.14666662954149, 0.800000011920929, 27.350476199672336, 26.82710286668528, 26.74716967288053, 26.996190487486974, 27.055238097054616, 26.897142826375507, 26.752381009147282, 27.215238137472245, 27.119811252040684, 26.94190479687282, 0.800000011920929, 27.27523808649608, 27.19716961541266, 27.502803640944936, 27.00095253898984, 27.123584868210667, 27.01214965982972, 27.252830236025577, 26.808411191, 26.980373808156664, 27.24433951332884, 27.413084281382158, 27.217924683161502, 26.773831717321805, 26.797169960332365, 27.2660377925297, 27.261320757978368, 27.081308434499757, 27.133962269661563, 27.266981217096436, 0.800000011920929, 27.08679234756614, 26.880373701313946, 27.189622653542823, 0.800000011920929, 26.98396229069188, 26.910280448811076, 27.149056575770647, 26.956074756996653, 27.024299183181512, 27.222429759591538, 27.20754723054058, 27.00747669077365, 26.990654090297557, 26.992523326494982, 26.991588737920065, 26.891509378293776, 26.87943919406873, 26.95233644001952, 26.881308464246377, 26.899065331320898, 26.937383119190965, 26.98317769587597, 26.98411220200708, 26.914018709525884, 26.91869155714445, 27.21775699831615, 27.218518593245083, 27.292523371282027, 0.800000011920929, 27.12336444186273, 27.068518630332417, 27.088784968741585, 27.143518302727628, 26.894392555005084, 27.087850380166667, 27.17169816201588, 0.8999999761581421, 27.023584826937263, 0.800000011920929, 27.1813084148915, 26.66915893666098, 38.78133322238922]

Calculate all value in Y period, Avg Power, Frezeer Temp, Fridge temp
In [187]:
Power_yyy = all_temp_data (no_cycle_y_period, Power_y)
print('Stable each cycle data ', Power_yyy)

power_avg_y =np.mean(Power_yyy[(length-8):(length-2)])
print('Average Power in Y Period :', power_avg_y)

Stable each cycle data  [39.429629604757565, 29.425892937396252, 27.902803723500153, 27.55047614177068, 27.080000027588436, 27.342307808307503, 27.295192254277374, 27.16952376252129, 26.97333328099478, 27.083809601409094, 27.340384670175037, 27.324761923154195, 1.5, 27.10666675908225, 27.063809573082697, 27.050476187183744, 27.21509427057122, 27.128571450710297, 27.413084031265473, 27.072380935010457, 27.12000009786515, 27.048113069444334, 27.311320766525448, 27.14666662954149, 0.800000011920929, 27.350476199672336, 26.82710286668528, 26.74716967288053, 26.996190487486974, 27.055238097054616, 26.897142826375507, 26.752381009147282, 27.215238137472245, 27.119811252040684, 26.94190479687282, 0.800000011920929, 27.27523808649608, 27.19716961541266, 27.502803640944936, 27.00095253898984, 27.123584868210667, 27.01214965982972, 27.252830236025577, 26.808411191, 26.980373808156664, 27.24433951332884, 27.413084281382158, 27.217924683161502, 26.773831717321805, 26.797169960332365, 27.2660377925297, 27.261320757978368, 27.081308434499757, 27.133962269661563, 27.266981217096436, 0.800000011920929, 27.08679234756614, 26.880373701313946, 27.189622653542823, 0.800000011920929, 26.98396229069188, 26.910280448811076, 27.149056575770647, 26.956074756996653, 27.024299183181512, 27.222429759591538, 27.20754723054058, 27.00747669077365, 26.990654090297557, 26.992523326494982, 26.991588737920065, 26.891509378293776, 26.87943919406873, 26.95233644001952, 26.881308464246377, 26.899065331320898, 26.937383119190965, 26.98317769587597, 26.98411220200708, 26.914018709525884, 26.91869155714445, 27.21775699831615, 27.218518593245083, 27.292523371282027, 0.800000011920929, 27.12336444186273, 27.068518630332417, 27.088784968741585, 27.143518302727628, 26.894392555005084, 27.087850380166667, 27.17169816201588, 0.8999999761581421, 27.023584826937263, 0.800000011920929, 27.1813084148915, 26.66915893666098, 38.78133322238922]
Average Power in Y Period : 18.360740295348396
In [188]:

# Power_y_plot
In [189]:

# Freezer Room average temperature in Y period 
Freezer_Temp_y = Freezer_Temp[ab:bb,]
freezer_temp_y = all_temp_data (no_cycle_y_period, Freezer_Temp_y)
print('Stable each cycle data frezzer room ', freezer_temp_y)

freezer_avg_y =np.mean(freezer_temp_y[(length-8):(length-2)])
print('Average Freezer in Y Period :', freezer_avg_y)

Stable each cycle data frezzer room  [-14.633185191473201, -18.677303622450147, -18.74852337703527, -18.747904792059035, -18.715523861476356, -18.720596163089457, -18.73017312930181, -18.70457143147786, -18.71184762137277, -18.72083810715448, -18.703211600963897, -18.705447616577146, -17.858000183105467, -18.69365717388334, -18.708857127598357, -18.663695266360328, -18.64630191191187, -18.604495208376935, -18.62525231192046, -18.642704792930967, -18.64207615988596, -18.61358490170173, -18.645584890977393, -18.63388567788261, -17.755999755859374, -18.623333342415947, -18.65829908424449, -18.659735864963174, -18.696895206996366, -18.702533313206256, -18.726971422831216, -18.752476192656015, -18.769009546552383, -18.76247169206727, -18.739447630019427, -17.8439998626709, -18.70038094111851, -18.68716978936825, -18.684112158445547, -18.69396188645136, -18.70201882236409, -18.72000002281688, -18.704056624646455, -18.692336444319974, -18.677476651200628, -18.703301892190616, -18.682411227716468, -18.71177358627319, -18.680112164934098, -18.700301886504544, -18.685113208698773, -18.694603781430224, -18.700448576312187, -18.706226426250527, -18.697547160454516, -17.820000076293944, -18.704018886134314, -18.71013078778704, -18.69826417239207, -17.79199981689453, -18.695150958367122, -18.68738319183064, -18.69473586172428, -18.710093487534568, -18.714691598838744, -18.706878467809375, -18.694698110616425, -18.694112123507214, -18.698018744281516, -18.675607460235884, -18.697495326817595, -18.726339622713486, -18.693719625027377, -18.714186925085908, -18.684934608958596, -18.70394394063504, -18.702542071476163, -18.717775747709183, -18.69323362546546, -18.681850438697314, -18.70265423070604, -18.699345793679502, -18.677129623625014, -18.691084109511337, -17.793999862670898, -18.701887891894188, -18.67490741058633, -18.703943924591922, -18.679240735371902, -18.733457921821373, -18.763943916392105, -18.772849070351075, -17.89399948120117, -18.756811309310624, -17.77999954223633, -18.72758880686536, -18.720616859364732, -18.49842663574219]
Average Freezer in Y Period : -18.44919868772611
In [190]:

# Fridge Room average temperature in X period 

Fridge_Temp_y=  Fridge_Temp[ab:bb,]
fridge_temp_y = all_temp_data (no_cycle_y_period, Fridge_Temp_y)
print('Stable each cycle data fridge room ', fridge_temp_y)

fridge_avg_y =np.mean(fridge_temp_y[(length-8):(length-2)])
print('Average Fridge in Y Period :', fridge_avg_y)

Stable each cycle data fridge room  [4.128909473311262, 3.5423809614564674, 3.373769470464403, 3.352253971402608, 3.382317455231198, 3.3945192239987554, 3.363846158178953, 3.38761904807318, 3.3946666660762967, 3.392444444648803, 3.4101602486692943, 3.389206353255682, 4.396666685740153, 3.3828253961744768, 3.3869841267192182, 3.4306031758823092, 3.4332389848412213, 3.4547301566790023, 3.451557624934246, 3.450476193617259, 3.433015865560563, 3.460849054204592, 3.445062891304867, 3.442666663063898, 4.506666819254558, 3.4371746106753283, 3.437694702935739, 3.426761006784139, 3.427809527752893, 3.39034920344277, 3.3840634919348216, 3.3758095319308943, 3.354190475978548, 3.361823906688571, 3.3873015895722407, 4.439999898274739, 3.4272380938605655, 3.4428301832211097, 3.4588473598905076, 3.421682545306191, 3.4277358441232897, 3.4292834874625537, 3.428899374772919, 3.454143312118506, 3.451713406036946, 3.431823901972682, 3.421370721308984, 3.4333647710722213, 3.46535825896486, 3.438396228559363, 3.444150937053393, 3.4254716957140263, 3.4150155730708, 3.4267610073464474, 3.446698105747595, 4.500000079472859, 3.435503136624329, 3.454766354085501, 3.443113200881946, 4.490000089009603, 3.43308175584805, 3.4230529626953246, 3.436603774252178, 3.4527102771949174, 3.443738304380317, 3.444361373828579, 3.450188678390575, 3.4264174436111694, 3.44495327468973, 3.4548286589506625, 3.447040498442369, 3.4331761056897028, 3.459813076387685, 3.4446417366232827, 3.46607477624097, 3.4642990699438285, 3.4608722858339833, 3.4566355241793336, 3.4644236752177324, 3.479688488063039, 3.4709657425078264, 3.464984424389039, 3.492407414464304, 3.470373838490044, 4.503333330154419, 3.475420571004862, 3.4879629751782355, 3.4774143467439673, 3.462067902824025, 3.413364490615988, 3.3788162007510087, 3.395880494665052, 4.429999987284343, 3.4158804988336255, 4.496666669845581, 3.427881623539969, 3.413208726224871, 3.2535111175643077]
Average Fridge in Y Period : 3.7575209124865965
In [191]:
y_time_End = no_cycle_y_period[-1]
print ('No of cycle period last time = ', no_cycle_y_period[-1]) 
y_time_Start = no_cycle_y_period[-8]
print ('No of cycle period first time= ', no_cycle_y_period[-8]) 
y_Time = (y_time_End - y_time_Start ) /120 
print ('Y Period in Hours  = ', y_Time      )

No of cycle period last time =  9599
No of cycle period first time=  9203
Y Period in Hours  =  3.3
In [192]:
#Plot Y period
plt.figure(figsize=(500, 100))
Power_y_plot = Power[y_time_Start:y_time_End]
plt.plot(Power_y_plot, color = 'r', linewidth = '10')
Out[192]:
[<matplotlib.lines.Line2D at 0x1bc80820b50>]

 

# Now calculated Other paramter between X - Y Period
In [193]:

# Energy (wh) value during the test period
Int_Energy =np.array(Int_Energy)
plt.plot(Int_Energy)
Out[193]:
[<matplotlib.lines.Line2D at 0x1bc80c5e610>]

 
In [194]:
X_time_End = x_time_End
X_time_Start = x_time_Start 
print ('Time at End up  X Period : ', x_time_End )
print ('Time at Start up  X Period : ', x_time_Start )

print ('Energy at End up  X Period : ', Int_Energy[x_time_End] )
print ('Energy at Start up  X Period : ', Int_Energy[x_time_Start] )

Time at End up  X Period :  1641
Time at Start up  X Period :  1243
Energy at End up  X Period :  368.2300109863281
Energy at Start up  X Period :  271.8999938964844
In [195]:
Y_time_End = y_time_End + aa
Y_time_Start = y_time_Start + aa
print ('Time at End up  y Period : ', Y_time_End )
print ('Time at Start up  y Period : ', Y_time_Start )

print ('Energy at End up  y Period : ', Int_Energy[Y_time_End] )
print ('Energy at Start up  y Period : ', Int_Energy[Y_time_Start] )

Time at End up  y Period :  11307
Time at Start up  y Period :  10911
Energy at End up  y Period :  2620.7000122070312
Energy at Start up  y Period :  2534.0000610351562

# Average Power , Temperature, other difference at end up X Period to end up Y period
In [196]:
#Average Power at end up  X Period to end up Y period
Power_avg_xy = np.mean(Power[X_time_End:Y_time_End])
print('Average Power at end up  X Period to end up Y period; ', Power_avg_xy)

Average Power at end up  X Period to end up Y period;  27.964411338417026
In [197]:

# Average Freezer Temperature at end up  X Period to end up Y period
Freezer_avg_xy = np.mean(Freezer_Temp[X_time_End:Y_time_End])

print('Average Freezer Temperature at end up  X Period to end up Y period; ', Freezer_avg_xy)

Average Freezer Temperature at end up  X Period to end up Y period;  -18.63547962436994
In [198]:
#Average Fridge Temperature at end up  X Period to end up Y period

Fridge_avg_xy = np.mean(Fridge_Temp[X_time_End:Y_time_End])

print('Average Fridge Temperature at end up  X Period to end up Y period; ', Fridge_avg_xy)

Average Fridge Temperature at end up  X Period to end up Y period;  3.4427763995108727
In [199]:

# Estimated Time at end up  X Period to end up Y period
print ('Time at End up  x Period : ', X_time_End )
print ('Time at End up  y Period : ', Y_time_End )

Est_time_xy =(Y_time_End - X_time_End ) /120
print ('Estimated Time at end up  X Period to end up Y period : ', Est_time_xy )

Time at End up  x Period :  1641
Time at End up  y Period :  11307
Estimated Time at end up  X Period to end up Y period :  80.55
In [200]:

# Estimated  Energy at end up  X Period to end up Y period
Est_Int_Energy_xy = Int_Energy[Y_time_End] - Int_Energy[x_time_End]
print ('Estimated  Energy at end up  X Period to end up Y period : ', Est_Int_Energy_xy    )

Estimated  Energy at end up  X Period to end up Y period :  2252.470001220703

Calculate the paramter for Period D and F
In [201]:

# D period taken from before of 1st defrost
no_cycle_d_period = no_cycle_x_period[-6:]
print('data ponit taken for D period: ' , no_cycle_d_period  )

data ponit taken for D period:  [1243, 1350, 1458, 1566, 1640, 1641]
In [202]:

# f period taken from before of 1st defrost
no_cycle_f_period = no_cycle_y_period[6:16] 
no_cycle_f_period = [ x + ab for x in no_cycle_f_period]
print('data ponit taken for D period: ' , no_cycle_f_period  )

data ponit taken for D period:  [2240, 2345, 2449, 2553, 2658, 2763]

# D & F Period calcuated, now need to calculated value Time , Energy, temperature
In [203]:

d_time_End = no_cycle_d_period[7]
print ('No of cycle period last time = ', no_cycle_d_period[7]) 
d_time_Start = no_cycle_d_period[2]
print ('No of cycle period first time= ', no_cycle_d_period[2]) 
d_Time = (d_time_End - d_time_Start ) /120 
print ('D Period in Hours  = ', d_Time      )



No of cycle period last time =  1641
No of cycle period first time=  1243
D Period in Hours  =  3.316666666666667
In [204]:
f_time_End = no_cycle_f_period[-1]
print ('No of cycle period last time = ', no_cycle_f_period[-1]) 
f_time_Start = no_cycle_f_period[-6]
print ('No of cycle period first time= ', no_cycle_f_period[-6]) 
f_Time = (f_time_End - f_time_Start ) /120 
print ('F Period in Hours  = ', f_Time      )

No of cycle period last time =  2763
No of cycle period first time=  2240
F Period in Hours  =  4.358333333333333

# Energy & Time in D & F Period
In [205]:
D_time_End = d_time_End
D_time_Start = d_time_Start

F_time_End = f_time_End
F_time_Start = f_time_Start

print ('Energy at End up of d Period : ', Int_Energy[D_time_End] )
print ('Energy at Start of  d Period : ', Int_Energy[D_time_Start] )
print ('Energy at End up of f Period : ', Int_Energy[F_time_End] )
print ('Energy at Start of  f Period : ', Int_Energy[F_time_Start] )

Energy at End up of d Period :  368.2300109863281
Energy at Start of  d Period :  271.8999938964844
Energy at End up of f Period :  688.2100219726562
Energy at Start of  f Period :  569.5499877929688

# Average power , Temperature , Energy in individual D & F periods
In [206]:
# Average Power in  D Period						
# Average Power in  F Period						
Power_avg_D = np.mean(Power[D_time_Start:D_time_End])
print('Average Power in  D Period; ', Power_avg_D)

Power_avg_F = np.mean(Power[F_time_Start:F_time_End])
print('Average Power in  F Period; ', Power_avg_F)

Average Power in  D Period;  29.00175880217672
Average Power in  F Period;  27.171510514292162
In [207]:
# Average Fridge Temperature in  D Period						
# Average Fridge Temperature  in  F Period						

Freezer_avg_D = np.mean(Freezer_Temp[D_time_Start:D_time_End])

print('Average Freezer Temperature in  D Period; ', Freezer_avg_D)

Freezer_avg_F = np.mean(Freezer_Temp[F_time_Start:F_time_End])

print('Average Freezer Temperature in  F Period; ', Freezer_avg_F)

Average Freezer Temperature in  D Period;  -18.63528641791799
Average Freezer Temperature in  F Period;  -18.716508627940783
In [208]:
# Average Freezer Temperature in  D Period						
# Average Freezer Temperature  in  F Period						

Fridge_avg_D = np.mean(Fridge_Temp[D_time_Start:D_time_End])

print('Average Freezer Temperature in  D Period; ', Fridge_avg_D)

Fridge_avg_F = np.mean(Fridge_Temp[F_time_Start:F_time_End])

print('Average Freezer Temperature in  F Period; ', Fridge_avg_F)

Average Freezer Temperature in  D Period;  3.433609710416596
Average Freezer Temperature in  F Period;  3.384614402545798

# Estimated Energy, Avg Temperature between period D & F
In [209]:
# Estimated  Energy at end up  D Period to end up F period

Est_Int_Energy_DF = Int_Energy[F_time_End] - Int_Energy[D_time_End]
print ('Estimated  Energy at end up  D Period to end up F period : ', Est_Int_Energy_DF    )

Est_Int_Energy_DsFe = Int_Energy[F_time_End] - Int_Energy[D_time_Start]
print ('Estimated  Energy at Start of  D Period to end up F period : ', Est_Int_Energy_DsFe    )

Estimated  Energy at end up  D Period to end up F period :  319.9800109863281
Estimated  Energy at Start of  D Period to end up F period :  416.3100280761719
In [210]:

# Estimated  Time at end up  D Period to end up F period


print ('Time at End up  F Period : ', F_time_End )
print ('Time at End up  D Period : ', D_time_End )

Est_time_df =(F_time_End - D_time_End ) /120
print ('Estimated Time at end up  D Period to end up F period  (hours): ', Est_time_df )

Time at End up  F Period :  2763
Time at End up  D Period :  1641
Estimated Time at end up  D Period to end up F period  (hours):  9.35
In [211]:
#Estimated  Average Freezer at end up  D Period to end up F period

Freezer_avg_DF = np.mean(Freezer_Temp[D_time_End:F_time_End])
print('Estimated  Average Freezer at end up  D Period to end up F period: ', Freezer_avg_DF)

Estimated  Average Freezer at end up  D Period to end up F period:  -18.225452787773428
In [212]:
#Estimated  Average Fridge at end up  D Period to end up F period

Fridge_avg_DF = np.mean(Fridge_Temp[D_time_End:F_time_End])

print('Average Freezer Temperature in  D Period; ', Fridge_avg_DF)

Average Freezer Temperature in  D Period;  3.4812240068996076
In [213]:
#Estimated  Average Power at   D Period &  F period
Power_avg_df = (Power_avg_D + Power_avg_F)/2

print('Estimated  Average Power at   D Period &  F period: ', Power_avg_df)

Estimated  Average Power at   D Period &  F period:  28.08663465823444
In [214]:
#Estimated Average Freezer Temperature at   D Period &  F period

Freezer_avg_df = (Freezer_avg_D + Freezer_avg_F) /2

print ('Estimated Average Freezer Temperature at   D Period &  F period :', Freezer_avg_df)

Estimated Average Freezer Temperature at   D Period &  F period : -18.675897522929386
In [215]:
#Estimated Average Fridge Temperature at  D Period &  F period

Fridge_avg_df = ( Fridge_avg_D + Fridge_avg_F )/2

print ('Estimated Average Fridge Temperature at  D Period &  F period; ', Fridge_avg_df)

Estimated Average Fridge Temperature at  D Period &  F period;  3.409112056481197
In [216]:
#Estimated  Time at   D Period Start to  F period end  

print ('Time at Sart of  D Period : ', D_time_Start )
print ('Time at End up  F Period : ', F_time_End )

Est_time_df =(F_time_End - D_time_Start ) /120
print ('Estimated  Time (hours) at   D Period Start to  F period end: ', Est_time_df )

Time at Sart of  D Period :  1243
Time at End up  F Period :  2763
Estimated  Time (hours) at   D Period Start to  F period end:  12.666666666666666

# Calculate factors Tdf , F_Tavg , R_Tavg and
# ∆Edf , ∆Thdf for F & R Compartment calculation
In [217]:
# Accumulated temperature difference over time in compartment F (∆Thdf_F)
del_Thdf_F =  Est_time_df *(Freezer_avg_DF - Freezer_avg_df)

print ('Accumulated temperature difference over time in compartment F (∆Thdf_F) : ' , del_Thdf_F)

Accumulated temperature difference over time in compartment F (∆Thdf_F) :  5.705633311975473
In [218]:
#Accumulated temperature difference over time in compartment R (∆Thdf_R)

del_Thdf_R =  Est_time_df *(Fridge_avg_DF - Fridge_avg_df)

print ('Accumulated temperature difference over time in compartment R (∆Thdf_R) : ' , del_Thdf_R)

Accumulated temperature difference over time in compartment R (∆Thdf_R) :  0.9134180386332018
In [219]:
# Additional energy consumed by the refrigerating appliance for defrost and recovery period (∆Edf)

del_Edf =    Est_Int_Energy_DsFe - (Power_avg_df *Est_time_df )  

print ('Additional energy consumed by the refrigerating appliance for defrost and recovery period (∆Edf) : ' , del_Edf)

Additional energy consumed by the refrigerating appliance for defrost and recovery period (∆Edf) :  60.54598907186897

# machine run % During stable period
In [233]:
# compressor run % calculated based on stable period (consider X  Period)
on_time, off_time =  on_off_cycle (Power_x_period)

run_per_comp = run_per (on_time, off_time)
comp_run_perct = np.mean (run_per_comp[5])
#comp_run_perct = 0.56
print('compressor run % calculated based on stable period', comp_run_perct)

compressor run % calculated based on stable period 0.5700934579439252

Tdf calculation based compresssor run time & other factor
In [234]:

# maximum possible defrost interval ∆tdmax 
del_tdmax = 39/comp_run_perct

# minimum possible defrost interval (∆tdmini)
del_tdmin = 6/comp_run_perct
In [235]:

# Actual defrost interval in data 
del_tdact = (Defrost_start[1] -Defrost_start[0] )/120
print ('Actual defrost interval in data  (∆tdact) : ' , del_tdact )

Actual defrost interval in data  (∆tdact) :  80.88333333333334
In [236]:
#Defrost interval for an ambient temperature of 32 °C (∆tdf)
del_tdf = (del_tdmax*del_tdmin)/(0.2*(del_tdmax-del_tdmin)+del_tdmin)
#del_tdf = 52
print('Defrost interval for an ambient temperature of 32 °C (∆tdf) : ', del_tdf)

Defrost interval for an ambient temperature of 32 °C (∆tdf) :  32.576112412177984
In [237]:

# Steady state power for the selected defrost control cycle (PSS2)
PSS2 =  (Est_Int_Energy_xy-del_Edf) / Est_time_xy

print ('Steady state power for the selected defrost control cycle (PSS2)' , PSS2)

Steady state power for the selected defrost control cycle (PSS2) 27.21196787273537

# Enrgy consumption daily and year (E_daily)
In [238]:
#Energy in Wh over a period of 24hr
E_daily = PSS2*24 + (del_Edf*24)/del_tdf

print ('Energy in Wh over a period of 24hr (E_daily)' , E_daily)


# Energy in Wh over a period of year (unit)
E_daily_year = E_daily*0.365

print ('Energy in Wh over a period of year (unit) (E_daily_year)' , E_daily_year)

Energy in Wh over a period of 24hr (E_daily) 697.6936485005384
Energy in Wh over a period of year (unit) (E_daily_year) 254.6581817026965

Average Temperature calculated in F & R Compartment
In [239]:

# steady state temperature in compartment F that occurs in the whole test period used for SS2 in degrees C;						
Tss2_F =  Freezer_avg_xy-   (del_Thdf_F/Est_time_xy)
print ('steady state temperature in compartment F that occurs in the whole test period used for SS2' , Tss2_F)

steady state temperature in compartment F that occurs in the whole test period used for SS2 -18.70631306089353
In [240]:
#steady state temperature in compartment R that occurs in the whole test period used for SS2 in degrees C;						
Tss2_R =  Fridge_avg_xy-   (del_Thdf_R/Est_time_xy)
print ('steady state temperature in compartment R that occurs in the whole test period used for SS2' , Tss2_R)

steady state temperature in compartment R that occurs in the whole test period used for SS2 3.4314366349095913
In [241]:
#T_favg	Average temperature for the compartment F over a complete defrost control cycle
T_favg = Tss2_F + (del_Thdf_F/del_tdf)
print ('Average temperature for the compartment F over a complete defrost control cycle :', T_favg)

Average temperature for the compartment F over a complete defrost control cycle : -18.531165294954384
In [242]:
#T_ravg	Average temperature for the compartment R over a complete defrost control cycle
T_ravg = Tss2_R + (del_Thdf_R/del_tdf)
print ('Average temperature for the compartment R over a complete defrost control cycle :', T_ravg)

Average temperature for the compartment R over a complete defrost control cycle : 3.4594761390430477

# We have to conculde our results based on defrost percentage
In [243]:
print('X-Y period Energy ', Est_Int_Energy_xy)
print('X-Y period Time', Est_time_xy)
print ('for more detail check upper code')
H24_energy  = Est_Int_Energy_xy*24/Est_time_xy
print('Daily Wh energy comsumption based derost to defrost data, consider one defrost average data in 24hr', H24_energy)
print ('Calculated Daily Wh energy comsumption based on all factors', E_daily)

X-Y period Energy  2252.470001220703
X-Y period Time 80.55
for more detail check upper code
Daily Wh energy comsumption based derost to defrost data, consider one defrost average data in 24hr 671.1270022259079
Calculated Daily Wh energy comsumption based on all factors 697.6936485005384
In [244]:

## Percentage increase in energy due to defrost factor
Percentage_increase_energy = (E_daily-H24_energy)*100 /H24_energy

print ('Percentage increase in energy due to defrost factor :',Percentage_increase_energy,'%' )

Percentage increase in energy due to defrost factor : 3.958512500095767 %
In [245]:
# Average Freezer and fridge Temperature increase 
print('Average Freezer Temperature at end up  X Period to end up Y period; ', Freezer_avg_xy)
print('Average Fridge Temperature at end up  X Period to end up Y period; ', Fridge_avg_xy)

# Calculated Average Freezer and fridge Temperature

print('Actual Freezer Temperature increase at end up  X Period to end up Y period; ', T_favg)
print('Actual Freezer Temperature increase at end up  X Period to end up Y period; ', T_ravg)

# Percentage change in F & R Temperature 
Percentage_increase_F_Temp = (Freezer_avg_xy-T_favg)*100 /Freezer_avg_xy
Percentage_increase_R_Temp = (T_ravg-Fridge_avg_xy)*100 /Fridge_avg_xy

print ('Percentage increase in Favg Temperature due to defrost factor :',Percentage_increase_F_Temp,'%' )
print ('Percentage increase in Favg Temperature due to defrost factor :',Percentage_increase_R_Temp,'%' )

Average Freezer Temperature at end up  X Period to end up Y period;  -18.63547962436994
Average Fridge Temperature at end up  X Period to end up Y period;  3.4427763995108727
Actual Freezer Temperature increase at end up  X Period to end up Y period;  -18.531165294954384
Actual Freezer Temperature increase at end up  X Period to end up Y period;  3.4594761390430477
Percentage increase in Favg Temperature due to defrost factor : 0.5597619783240846 %
Percentage increase in Favg Temperature due to defrost factor : 0.4850660511832155 %
In [ ]:
 


# Value calculation for Data export to excel
In [177]:



In [178]:


plt.figure(figsize=(20, 10))
plt.plot(Power)


Out[178]:
[<matplotlib.lines.Line2D at 0x1ca030dc610>]



X & Y Period Power and Temperature
In [179]:


## X & Y Period length 
print (' X period Start time = ', X_time_Start) 
print (' X period End time = ', X_time_End  ) 
​
print (' Y period Start time = ',Y_time_Start ) 
print (' Y period End time = ', Y_time_End )
​
​
X_Time = (X_time_End - X_time_Start ) /120 
print ('X Period in Hours  = ', X_Time  )
​
Y_Time = (Y_time_End - Y_time_Start ) /120 
print ('Y Period in Hours  = ', Y_Time )



 X period Start time =  1155
 X period End time =  1984
 Y period Start time =  10921
 Y period End time =  11745
X Period in Hours  =  6.908333333333333
Y Period in Hours  =  6.866666666666666
In [180]:


## Period wise Freezer & Fride average data 
​
#Average Power at  X Period 
Power_avg_xx = np.mean(Power[X_time_Start:X_time_End])
print('Average Power at   X Period ; ', Power_avg_xx)
​
​
# Average Freezer Temperature  X Period 
Freezer_avg_xx = np.mean(Freezer_Temp[X_time_Start:X_time_End])
​
print('Average Freezer Temperature at  X Period ; ', Freezer_avg_xx)
​
Fridge_avg_xx = np.mean(Fridge_Temp[X_time_Start:X_time_End])
​
print('Average Fridge Temperature at  X Period; ', Fridge_avg_xx)



Average Power at   X Period ;  25.793486121911624
Average Freezer Temperature at  X Period ;  -18.41783833912112
Average Fridge Temperature at  X Period;  3.5977804641623905
In [181]:


## Period wise Freezer & Fride average data 
​
#Average Power at  X Period 
Power_avg_yy = np.mean(Power[Y_time_Start:Y_time_End])
print('Average Power at   Y Period ; ', Power_avg_yy)
​
# Average Freezer Temperature  Y Period 
Freezer_avg_yy = np.mean(Freezer_Temp[Y_time_Start:Y_time_End])
​
print('Average Freezer Temperature at  Y Period ; ', Freezer_avg_yy)
​
Fridge_avg_yy = np.mean(Fridge_Temp[Y_time_Start:Y_time_End])
​
print('Average Fridge Temperature at  Y Period; ', Fridge_avg_yy)



Average Power at   Y Period ;  25.92985440320471
Average Freezer Temperature at  Y Period ;  -18.358373772750777
Average Fridge Temperature at  Y Period;  3.63692152808785
In [182]:


print ('Energy at End up  X Period : ', Int_Energy[X_time_End] )
print ('Energy at Start up  X Period : ', Int_Energy[X_time_Start] )
print ('Energy at End up  y Period : ', Int_Energy[Y_time_End] )
print ('Energy at Start up  y Period : ', Int_Energy[Y_time_Start] )



Energy at End up  X Period :  423.8700256347656
Energy at Start up  X Period :  246.00003051757812
Energy at End up  y Period :  2600.0799560546875
Energy at Start up  y Period :  2421.909957885742

# F & D Period Power and Temperature
In [183]:


print ('Energy at End up of d Period : ', Int_Energy[D_time_End] )
print ('Energy at Start of  d Period : ', Int_Energy[D_time_Start] )
print ('Energy at End up of f Period : ', Int_Energy[F_time_End] )
print ('Energy at Start of  f Period : ', Int_Energy[F_time_Start] )



Energy at End up of d Period :  373.0199890136719
Energy at Start of  d Period :  246.00003051757812
Energy at End up of f Period :  882.9199523925781
Energy at Start of  f Period :  754.4199523925781
In [184]:


## Period wise Freezer & Fride average data 
​
#Average Power at  X Period 
Power_avg_dd = np.mean(Power[D_time_Start:D_time_End])
print('Average Power at   D Period ; ', Power_avg_dd)
​

# Average Freezer Temperature F Period 
Freezer_avg_dd = np.mean(Freezer_Temp[D_time_Start:D_time_End])
​
print('Average Freezer Temperature at  D Period ; ', Freezer_avg_dd)
​
Fridge_avg_dd = np.mean(Fridge_Temp[D_time_Start:D_time_End])
​
print('Average Fridge Temperature at  D Period; ', Fridge_avg_dd)



Average Power at   D Period ;  25.816694769951788
Average Freezer Temperature at  D Period ;  -18.418711615654093
Average Fridge Temperature at  D Period;  3.598611584401252
In [185]:


##  Period wise Freezer & Fride average data 
​
#Average Power at  X Period 
Power_avg_ff = np.mean(Power[F_time_Start:F_time_End])
print('Average Power at   F Period ; ', Power_avg_ff)
​
#  Average Freezer Temperature F Period 
Freezer_avg_ff = np.mean(Freezer_Temp[F_time_Start:F_time_End])
​
print('Average Freezer Temperature at  F Period ; ', Freezer_avg_ff)
​
Fridge_avg_ff = np.mean(Fridge_Temp[F_time_Start:F_time_End])
​
print('Average Fridge Temperature at  F Period; ', Fridge_avg_ff)



Average Power at   F Period ;  26.24863937497139
Average Freezer Temperature at  F Period ;  -18.426197280364793
Average Fridge Temperature at  F Period;  3.5820351527526526
In [186]:


# D & F Period length 
print (' D period Start time = ', D_time_Start) 
print (' D period End time = ', D_time_End  ) 
​
print (' F period Start time = ',F_time_Start ) 
print (' F period End time = ', F_time_End )
​
​
D_Time = (D_time_End - D_time_Start ) /120 
print ('D Period in Hours  = ', D_Time  )
​
F_Time = (F_time_End - F_time_Start ) /120 
print ('Y Period in Hours  = ', F_Time )



D period Start time =  1155
D period End time =  1748
F period Start time =  3211
F period End time =  3799
D Period in Hours  =  4.941666666666666
Y Period in Hours  =  4.9
In [187]:


# Estimated  Average Power at   D Period &  F period
Power_avg_xy_e = (Power_avg_xx + Power_avg_yy)/2
​
print('Estimated  Average Power at   X Period &  Y period: ', Power_avg_xy_e)



Estimated  Average Power at   X Period &  Y period:  25.861670262558167
In [188]:




# Estimated  Average Power at   D Period &  F period
Power_avg_df_e = (Power_avg_D + Power_avg_F)/2
​
print('Estimated  Average Power at   D Period &  F period: ', Power_avg_df_e)



Estimated  Average Power at   D Period &  F period:  26.03266707246159
In [189]:


# time Ration for X-Y and D-f period
​
Time_ratio_xy = X_Time/Y_Time
Time_ratio_df = D_Time/F_Time
​
print('Estimated time ration between X and Y Period : ', Time_ratio_xy)
print('Estimated time ration between D and F Period : ', Time_ratio_df)
​



Estimated time ration between X and Y Period :  1.0060679611650485
Estimated time ration between D and F Period :  1.008503401360544
In [190]:

## Requirement for Spared check 
 Tspread_F  °C  0.5 °C  0.06        0.04    
 Tspread_R  °C  0.5 °C  0.05        0.09    
 P_spread(%)    %   < 2%    0.00        0.01    
 P_spread(W)    W   1W  0.06        0.29    
​




# Power and Temperature spread check
In [191]:


# P_spread(W)    W   1W  0.06        0.29    
​
Power_temp_spread_xy_e= np.abs(Power_avg_xx - Power_avg_yy)
Power_temp_spread_df_e= np.abs(Power_avg_dd - Power_avg_ff)
print ('Power spread in X & Y Period   = ', Power_temp_spread_xy_e )
print ('Power spread in D & F Period   = ', Power_temp_spread_df_e )
​



Power spread in X & Y Period   =  0.13636828129308753
Power spread in D & F Period   =  0.4319446050196021
In [192]:


# P_spread(%)    %   < 2%    0.00        0.01    
Power_temp_spread_xy_t= (np.abs(Power_avg_xx - Power_avg_yy)/Power_avg_xy_e)*100
Power_temp_spread_df_t= (np.abs(Power_avg_dd - Power_avg_ff)/Power_avg_df_e)*100
print ('Power spread in X & Y Period in pecentage   = ', Power_temp_spread_xy_t,'%')
print ('Power spread in D & F Period in percentage  = ', Power_temp_spread_df_t,'%' )



Power spread in X & Y Period in pecentage   =  0.5272988167764163 %
Power spread in D & F Period in percentage  =  1.659240691002923 %
In [193]:


Freezer_temp_spread_xy= np.abs(Freezer_avg_xx - Freezer_avg_yy)
Freezer_temp_spread_df= np.abs(Freezer_avg_dd - Freezer_avg_ff)
​
​
print ('Freezer Temperature spread in X & Y Period   = ', Freezer_temp_spread_xy )
print ('Freezer Temperature spread in D & F Period   = ', Freezer_temp_spread_df )



Freezer Temperature spread in X & Y Period   =  0.05946456637034103
Freezer Temperature spread in D & F Period   =  0.00748566471069978
In [194]:


Fridge_temp_spread_xy= np.abs(Fridge_avg_xx - Fridge_avg_yy)
Fridge_temp_spread_df= np.abs(Fridge_avg_dd - Fridge_avg_ff)
​
​
print ('Fridge Temperature spread in X & Y Period   = ', Fridge_temp_spread_xy )
print ('Fridge Temperature spread in D & F Period   = ', Fridge_temp_spread_df )



Fridge Temperature spread in X & Y Period   =  0.03914106392545946
Fridge Temperature spread in D & F Period   =  0.016576431648599232
In [195]:


## X and Y Temperature Value average individual period 
Freezer_avg_xy_e = (Freezer_avg_xx +Freezer_avg_yy)/2
​
print('Average Freezer Temperature at  X Period and Y Period ; ', Freezer_avg_xy_e)
​
Fridge_avg_xy_e  =  (Fridge_avg_xx + Fridge_avg_yy)/2
​
print('Average Fridge Temperature at  X Period and Y Period ; ', Fridge_avg_xy_e)
​



Average Freezer Temperature at  X Period and Y Period ;  -18.38810605593595
Average Fridge Temperature at  X Period and Y Period ;  3.6173509961251202
In [196]:


## D and F Temperature Value average individual period 
Freezer_avg_df_e = (Freezer_avg_dd +Freezer_avg_ff)/2
​
print('Average Freezer Temperature at  D Period and F Period ; ', Freezer_avg_df_e)
​
Fridge_avg_df_e  =  (Fridge_avg_dd + Fridge_avg_ff)/2
​
print('Average Fridge Temperature at D Period and F Period ; ', Fridge_avg_df_e)



Average Freezer Temperature at  D Period and F Period ;  -18.422454448009443
Average Fridge Temperature at D Period and F Period ;  3.5903233685769522
In [197]:


## X end to Y end Temperature Value 
​
​
# Average Freezer Temperature X End to Y end 
Freezer_avg_xy_t = np.mean(Freezer_Temp[X_time_End:Y_time_End])
​
print('Average Freezer Temperature at  X Period End  to Y Period End  ; ', Freezer_avg_xy_t)
​
Fridge_avg_xy_t = np.mean(Fridge_Temp[X_time_End:Y_time_End])
​
print('Average Fridge Temperature at  X Period End to Y Period End  ; ', Fridge_avg_xy_t)



Average Freezer Temperature at  X Period End  to Y Period End  ;  -18.2770232418864
Average Fridge Temperature at  X Period End to Y Period End  ;  3.622173962373896
In [198]:




## D Start to F end Temperature Value 
​
# Average Freezer Temperature D Start to F end 
Freezer_avg_df_t = np.mean(Freezer_Temp[D_time_Start:F_time_End])
​
print('Average Freezer Temperature at  D Period Start to F Period End  ; ', Freezer_avg_df_t)
​
Fridge_avg_df_t = np.mean(Fridge_Temp[D_time_Start:F_time_End])
​
print('Average Fridge Temperature at  D Period Start to F Period End  ; ', Fridge_avg_df_t)



Average Freezer Temperature at  D Period Start to F Period End  ;  -17.93484870665122
Average Fridge Temperature at  D Period Start to F Period End  ;  3.6814699992124127
In [199]:


print('compressor run % calculated based on stable period', comp_run_perct)



compressor run % calculated based on stable period 0.5913043478260869
In [200]:


# Energy simulation calculation 
Energy_Compensation= ((-18-T_favg)*0.035 + (4-T_ravg)*0.015)
Energy_Compensation = (1-Energy_Compensation)
Energy_simulation = Energy_Compensation*E_daily_year
print('Simulated energy based on Target Temperature (-18/4) :    ', Energy_simulation)



Simulated energy based on Target Temperature (-18/4) :     242.41397846328462

All Result value export to excel
In [205]:


Reults_data = {'Description': ['Freezer Avg','Fridge Avg','Energy Start','Energy End', 'Time(hr)', 'Power Avg(W)' , 
                              'Power_avg_XY, DF', 'Time Start', 'Time End', 'Time Diff: startD_endF//endX_endY',
                               'Tspread_F', 'Tspread_R',
                              'P_spread(%)', 'P_spread(W)', 'Time_Ratio', 'Tavg.F (X & Y // D & F)',
                               'Tavg.R (X & Y // D & F)', 'Tavg Freez.:StartD_endF//endX_end Y',
                              'Tavg Fridge:StartD_endF//endX_end Y', 'Pss2', '△Edfj', 'ΔTh df_F', 'ΔTh df_R',
                              'Tss2_F','Tss2_R','Taverage_Freezer', 'Taverage_Fridge', 'Comp Run Percentage %',
                              '△td-Max', '△td-Min', '△tdf', 'E_daily', 'Yearly Energy Consumption(KWH/Yr)', 
                               'Simulated Energy at (-18/4)'],
               
               
            'Unit/Spec' : ['°C','°C','Wh', 'Wh', 'X/Y>4hrs D/F>3hrs','W','W','Sec','Sec','°C','0.5°C','0.5°C',
                            '<2%','<1W','0.8-1.2','°C','°C','°C','°C','W',
                         'Wh','°C','°C','°C','°C','°C','°C','%','Hr','Hr','Hr','Wh/day','Kwh/Yr','Kwh/Yr'], 
               
            'X Period' : [Freezer_avg_xx,Fridge_avg_xx,Int_Energy[X_time_Start],Int_Energy[X_time_End],X_Time,
                          Power_avg_xx,Power_avg_xy_e,X_time_Start,X_time_End,Est_time_xy,Freezer_temp_spread_xy,
                          Fridge_temp_spread_xy,Power_temp_spread_xy_t,Power_temp_spread_xy_e, Time_ratio_xy,Freezer_avg_xy_e,
                          Fridge_avg_xy_e, Freezer_avg_xy_t, Fridge_avg_xy_t,PSS2,
                         del_Edf,del_Thdf_F,del_Thdf_R,Tss2_F,Tss2_R, T_favg, T_ravg,comp_run_perct*100,
                          del_tdmax,del_tdmin,del_tdf,E_daily,E_daily_year,Energy_simulation],
               
            'Y Period' : [Freezer_avg_yy,Fridge_avg_yy,Int_Energy[Y_time_Start],Int_Energy[Y_time_End],Y_Time,
                          Power_avg_yy," ",Y_time_Start,Y_time_End," "," "," "," "," ", " "," "," "," "," "," ",
                         " "," "," "," "," "," "," "," "," "," "," "," "," "," "],
               
            'D Period' : [Freezer_avg_dd,Fridge_avg_dd,Int_Energy[D_time_Start],Int_Energy[D_time_End],D_Time,
                          Power_avg_dd,Power_avg_df_e,D_time_Start,D_time_End,Est_time_df,Freezer_temp_spread_df,
                          Fridge_temp_spread_df,Power_temp_spread_df_t,Power_temp_spread_df_e, Time_ratio_df,Freezer_avg_df_e,
                          Fridge_avg_df_e,Freezer_avg_df_t, Fridge_avg_df_t," ",
                         " "," "," "," "," "," "," "," "," "," "," "," "," "," "],
               
            'F Period' : [Freezer_avg_ff,Fridge_avg_ff,Int_Energy[F_time_Start],Int_Energy[F_time_End],F_Time,
                          Power_avg_ff," ",F_time_Start,F_time_End," "," "," "," "," ", " "," "," "," "," "," ",
                         " "," "," "," "," "," "," "," "," "," "," "," "," "," "],
         }
​
R_data = pd.DataFrame(Reults_data, columns = ['Description', 'Unit/Spec', 'X Period', 'Y Period', 'D Period', 'F Period'])
​
print (R_data)



                            Description          Unit/Spec     X Period  \
0                           Freezer Avg                 °C   -18.417838   
1                            Fridge Avg                 °C     3.597780   
2                          Energy Start                 Wh   246.000031   
3                            Energy End                 Wh   423.870026   
4                              Time(hr)  X/Y>4hrs D/F>3hrs     6.908333   
5                          Power Avg(W)                  W    25.793486   
6                      Power_avg_XY, DF                  W    25.861670   
7                            Time Start                Sec  1155.000000   
8                              Time End                Sec  1984.000000   
9     Time Diff: startD_endF//endX_endY                 °C    81.341667   
10                            Tspread_F              0.5°C     0.059465   
11                            Tspread_R              0.5°C     0.039141   
12                          P_spread(%)                <2%     0.527299   
13                          P_spread(W)                <1W     0.136368   
14                           Time_Ratio            0.8-1.2     1.006068   
15              Tavg.F (X & Y // D & F)                 °C   -18.388106   
16              Tavg.R (X & Y // D & F)                 °C     3.617351   
17  Tavg Freez.:StartD_endF//endX_end Y                 °C   -18.277023   
18  Tavg Fridge:StartD_endF//endX_end Y                 °C     3.622174   
19                                 Pss2                  W    25.975328   
20                                △Edfj                 Wh    63.333491   
21                             ΔTh df_F                 °C    13.825998   
22                             ΔTh df_R                 °C     2.536108   
23                               Tss2_F                 °C   -18.446998   
24                               Tss2_R                 °C     3.590995   
25                     Taverage_Freezer                 °C   -18.037904   
26                      Taverage_Fridge                 °C     3.666036   
27                Comp Run Percentage %                  %    59.130435   
28                              △td-Max                 Hr    80.983333   
29                              △td-Min                 Hr    10.147059   
30                                 △tdf                 Hr    33.796662   
31                              E_daily             Wh/day   668.382830   
32    Yearly Energy Consumption(KWH/Yr)             Kwh/Yr   243.959733   
33          Simulated Energy at (-18/4)             Kwh/Yr   242.413978   

       Y Period    D Period    F Period  
0    -18.358374  -18.418712  -18.426197  
1      3.636922    3.598612    3.582035  
2   2421.909958  246.000031  754.419952  
3   2600.079956  373.019989  882.919952  
4      6.866667    4.941667         4.9  
5     25.929854   25.816695   26.248639  
6                 26.032667              
7         10921        1155        3211  
8         11745        1748        3799  
9                 22.033333              
10                 0.007486              
11                 0.016576              
12                 1.659241              
13                 0.431945              
14                 1.008503              
15               -18.422454              
16                 3.590323              
17               -17.934849              
18                  3.68147              
19                                       
20                                       
21                                       
22                                       
23                                       
24                                       
25                                       
26                                       
27                                       
28                                       
29                                       
30                                       
31                                       
32                                       
33                                       
In [207]:



Result_data = R_data.to_excel(r'D:\HAIL_DATA-1\E drive Data\2022\New Energy Norms _2023\Final.xlsx', sheet_name='final', index = False)



In [ ]:


