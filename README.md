# Tensile-Testing-Research-Custom-Octave-Code
Custom Octave code I developed for an extracurricular tensile testing research project. It can parse the data in an excel file, from a tensile testing machine. Then, it will plot the data to a stress strain curve, and to an elastic modulus.

%Tensile Testing Program
clc, clear;
data = input('Input your file name: ', 's'); %Have file prompt user to put in file name
fprintf('\n');%Create a new line
pkg load io; %Tell file to read
Tensile_Data = xlsread(data); %Extract data from file

n = input("Enter the number of samples you would like to calculate for: "); %Prompt user to enter number of vectors
length = input('Input your length for the samples (57 mm default): '); %Establish initial length
st = [1:n]; %create array for start times
en = [1:n]; %create array for end times
width=[1:n]; %create array for width
thickness=[1:n]; %create array for thickness
color1 = ['+b'; '+k'; '+r'; '+g'; '+m'; '+c']; %Create array for colors
color2 = ['b-'; 'k-'; 'r-'; 'g-'; 'm-'; 'c-']; %Create array for colors
Avg_Max_Strain = 0; %set for later

for i = 1:n %Create a for loop

  width(i) = Tensile_Data(i, 7); %Prompt a width input
  thickness(i) = Tensile_Data(i, 8); %Prompt a thickness input
  if (i<n)
  rctch= Tensile_Data(:, 1);
  [a_st, b_st] = find(rctch == i);
  k = a_st(2, 1) + 2;
  st(i)= k(1, 1);  %Have starting row be input
  [a_en, b_en] = find(rctch == i+1); 
  en(i)= a_en(2, 1) - 2; %Have ending row be input
elseif (i==n)
  rctch= Tensile_Data(:, 1);
  [a_st, b_st] = find(rctch == i);
  st(i)= a_st(2, 1) + 2; %Have starting row be input
  [a_en, b_en] = find(rctch);
  fen = max(a_en);
  en(i)=fen; %Have ending row be input
  endif

  area = width(i).*thickness(i); %Calculate area from width and thickness input


  displacement= Tensile_Data(st(i):en(i), 3); %Create a vector for displacement
  forceKN = Tensile_Data(st(i):en(i), 4); %Create a vector for force
  forceN = forceKN*1000; %Convert force from kilonewtons to newtons
  strain = displacement./length; %Create a vector for Strain
  stress = forceN./area;  %Create a vector for Stress

  if (i<n)
    plot(strain, stress, color1(i)); %Create a plot, for stress vs strain
    set(gca, 'fontsize', 20) %Alter font
    title('Stress Vs. Strain'); %Give title
    ylabel('Stress (Mpa)'); %Label y-axis
    xlabel('Strain'); %Label x-axis
    hold on
  elseif (i == n)
    plot(strain, stress, color1(i)); %Create a plot, for stress vs strain
    set(gca, 'fontsize', 20) %Alter font
    title('Stress Vs. Strain'); %Give title
    ylabel('Stress (Mpa)'); %Label y-axis
    xlabel('Strain'); %Label x-axis
    hold on
    set(gca, 'fontsize', 20)
    legend('Sample 1', 'Sample 2', 'Sample 3', 'Sample 4', 'Sample 5','Sample 6', 'Location', 'northeastoutside')
  endif
end

string1 = 'done'; %Set string value, for if else
fprintf('\nPlease save, then close, your current ''Stress vs. Strain'' plot.\n'); %Prompt user to save their first plot
string2 = input('Type ''done'', when ready for the ''Linear Regression'' plot: '  ,'s'); %Prompt user to input 'done', for if else

if (strcmp(string1, string2) == 1) %Use if else to create plot
for i = 1:n %Create a for loop
 
  time = Tensile_Data(st(i):en(i), 2); %Create a vector for time
  displacement= Tensile_Data(st(i):en(i), 3); %Create a vector for displacement
  forceKN = Tensile_Data(st(i):en(i), 4); %Create a vector for force
  forceN = forceKN*1000; %Convert force from kilonewtons to newtons
  strain = displacement./length; %Create a vector for Strain
  stress = forceN./area;  %Create a vector for Stress
     
  locs = find(stress>=5 & stress<=10); %Find locations where stress is between 5 and 10 Mpa
  elastress = stress(locs); %Calculate stress values, for said locations
  elastrain = strain(locs); %Calculate corresponding stress values
  [a1, b1] = polyfit(elastrain, elastress, 1); %Find line regression of best fit, via polyfit
  m_eq = a1(1, 1); %Find elastic modulus scalar/slope via polyfit
  b_eq = a1(1, 2); %Find b for line's equation 
  y = m_eq.*elastrain + b_eq; %Find y, for line of best fit 

  m_st = num2str(m_eq); %Turn numerical slope into string
  b_st = num2str(b_eq); %Turn numerical b into string
  ystring = 'Y = '; %Make first component of linear regression equation
  plusstring = '*X + '; %Make second component of linear regression equation
  Linear_Regression = [ystring, m_st, plusstring, b_st]; %Combine all components, for linear regression
  
  if (i<n)
  plot(elastrain, elastress, color1(i)); %Create a plot, for between 5 and 10 Mpa
  set(gca, 'fontsize', 20) %Alter font
  title('Linear Regression'); %Give title
  ylabel('Stress (Mpa)'); %Label y-axis
  xlabel('Strain'); %Label x-axis
  hold on %Have file hold on to plot
  plot(elastrain, y, color2(i)) %Plot the linear regression
  axis([0.006, 0.022, 5, 11])
  hold on
  set(gca, 'fontsize', 20)
  text(0.006, (10.99999-(0.3*i)), Linear_Regression, 'fontsize', 14) %Put Linear Regression equation in plot
  elseif (i==n)
  plot(elastrain, elastress, color1(i)); %Create a plot, for between 5 and 10 Mpa
  set(gca, 'fontsize', 20) %Alter font
  title('Linear Regression'); %Give title
  ylabel('Stress (Mpa)'); %Label y-axis
  xlabel('Strain'); %Label x-axis
  hold on %Have file hold on to plot
  plot(elastrain, y, color2(i)) %Plot the linear regression
  axis([0.006, 0.022, 5, 11])
  hold on
  set(gca, 'fontsize', 15)
  legend('Stress Vs. Strain', 'Linear Regression', 'Location', 'Southeast') %Give legend
  text(0.006, (10.99999-(0.3*i)), Linear_Regression, 'fontsize', 14) %Put Linear Regression equation in plot
  legend('Stress vs. Strain 1', 'Linear Regression 1', 'Stress vs. Strain 2', 'Linear Regression 2','Stress vs. Strain 3', 'Linear Regression 3','Stress vs. Strain 4', 'Linear Regression 4','Stress vs. Strain 5', 'Linear Regression 5', 'Stress vs. Strain 6', 'Linear Regression 6', 'Location', 'northeastoutside');
  axis([0.006, 0.022, 5, 11])
  endif

  fprintf('\n');%Create a new line
  fprintf('Linear Regression %d: ', i);
  fprintf(Linear_Regression);%Output Linear Regression to Window
  fprintf('\n \n'); %Create a new line

  elastic_modulus = m_eq; %Create elastic modulus scalar
  uts = max(stress); %Create a scalar for UTS
  maxstrain(i) = Tensile_Data(i, 4)/57; %Create scalar for Max Strain

  locs2 = find(stress>=(uts*0.59) & stress<=(uts*0.61)); %Find locations where stress is between 0.59 UTS and 0.61 UTS
  strain_elong = strain(locs2); %Calculate strain values, for said locations
  elongation = max(strain_elong); %Create a scalar for elongation, using max strain value 
  percent_elong = elongation*100; %Create a scalar for percent elongation

  fprintf('\nTable %d:\n', i)
  Names = ['UTS (MPa)'; 'Percent Elongation'; 'Elongation'; 'Elastic Modulus (MPa)'; 'Max Strain']; %Give names, for header
  Tensile_Table_Data = [time, displacement, forceKN, forceN, strain, stress]; %Create table for raw data
  Tensile_Table_Calcs = [uts; percent_elong; elongation; elastic_modulus; maxstrain(i)]; %Create table for calculations
  string_calcs = num2str(Tensile_Table_Calcs); %Convert calculations to strings
  Tensile_Table_Main = [Names, string_calcs] %Output table
  
  Avg_Max_Strain = Avg_Max_Strain + maxstrain(i);
end

elseif (strcmp(string1, string2) == 0)
fprintf('Error.'); %In case wrong string was typed in

endif
fprintf('\n\n')
Avg_Max_Strain = Avg_Max_Strain/n
