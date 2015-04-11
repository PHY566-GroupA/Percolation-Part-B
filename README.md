# Percolation-Part-B
part_1

from pylab import *
import random
from time import time

n=100   #linear dimension of grid (n*n)
#N=20  #number of distinct probability values
#probability=linspace(0.593,0.750,N)
#iterations=50   #number of iterations for each probability value
p=0.599 #use this probability value for the time being
grid = zeros((n,n))
coordinate=[]
cluster_dictionary={1:[]}  #stores the coordinates [x,y] of every element in a particular cluster.. and this is a dictionary of all the possible clusters,keys numbered starting from 1; gets dynamically updated.
time1=time()
def lies(x,cluster_dictionary):
        val=0
	for i in range(len(cluster_dictionary)):     #search different clusters
		for j in range(len(cluster_dictionary[i+1])):   #search different elements within particular cluster
			if x==cluster_dictionary[i+1][j]:
                                val=i+1
                                break
        return val       

count=0	#to count the number of filled sites in the grid		
for i in range(n):
	for j in range(n):
		r=random.random()
		if r<p:
                        count+=1
			grid[i,j]=1 #occupy the site with a probability p
			coordinate.append([i,j])

temp=[]   #temporary array containing the coordinates we'll be working with
for i in range(len(coordinate)):
	temp.append(coordinate[i])
# otherwise setting temp=coordinate and later deleting elements from temp, deletes elements from coordinate as well
	
cluster_dictionary[1].append(temp[0]) #append the first occupied site coordinate to the first cluster	
temp.pop(0)

#temp[0] is the last element we are checking for
for l in range(len(coordinate)-1): #excluding the first element, that has already been taken care of
	a=0 #keeps count of the number of empty adjacent sites
	b=0 #keeps count of the number of adjacent sites
	z=[0,0,0,0]
	if (temp[0][0]+1)<n: #i.e, the right adjacent x coordinate lies within the grid 
		b+=1
		if lies([temp[0][0]+1,temp[0][1]],cluster_dictionary)!=0:  #(adjacent one lies in some cluster)..also !=0 works because no cluster has the key 0 in the dictionary. the keys start from 1.
			z[0]=lies([temp[0][0]+1,temp[0][1]],cluster_dictionary)
			cluster_dictionary[z[0]].append(temp[0])
		else:
			a+=1  #this adjacent one is empty
	
	if (temp[0][0]-1)>=0: #i.e the left adjacent x coordinate lies within the grid
		b+=1
		if lies([temp[0][0]-1,temp[0][1]],cluster_dictionary)!=0:  #(adjacent one lies in some cluster)
			z[1]=lies([temp[0][0]-1,temp[0][1]],cluster_dictionary)
			if temp[0] not in cluster_dictionary[z[1]]: #necessary as it might already be 
				cluster_dictionary[z[1]].append(temp[0])
		else:
			a+=1  #this adjacent one is empty
#trouble here
	if (temp[0][1]+1)<n: #i.e the upper adjacent y coordinate lies within the grid 
		b+=1
		if lies([temp[0][0],temp[0][1]+1],cluster_dictionary)!=0:  #(adjacent one lies in some cluster)
			z[2]=lies([temp[0][0],temp[0][1]+1],cluster_dictionary)
			if temp[0] not in cluster_dictionary[z[2]]:
				cluster_dictionary[z[2]].append(temp[0])
		else:
			a+=1  #this adjacent one is empty

	if (temp[0][1]-1)>=0: #i.e the lower adjacent y coordinate lies within the grid
		b+=1
		if lies([temp[0][0],temp[0][1]-1],cluster_dictionary)!=0:  #(adjacent one lies in some cluster)
			z[3]=lies([temp[0][0],temp[0][1]-1],cluster_dictionary)
			if temp[0] not in cluster_dictionary[z[3]]:
				cluster_dictionary[z[3]].append(temp[0])
		else:
			a+=1  #this adjacent one is empty

	if a==b:  #the number of adjacent sites matches the number of empty adjacent sites
		r=len(cluster_dictionary)+1  
		cluster_dictionary[r]=[]   #numbering of clusters start from 1.adding a new cluster here with this latest unused coordinate(as it has no adjacent filled sites), being put in that new cluster
		cluster_dictionary[r].append(temp[0])

#now need to merge ALL( at most 4) clusters in which temp[0] may have entered, if it didnt already create a separate cluster for itself when a==4 above.
	temp_list=cluster_dictionary.keys()   #temporary list of the keys in cluster_dictionary
	indices=[]      #stores the key values of the clusters that contain temp[0]
	for i in range(len(temp_list)):    #run over all the clusters
		if temp[0] in cluster_dictionary[temp_list[i]]: #or cudve just written if temp[0] in cluster_dictionary[temp_list[i]]
			indices.append(temp_list[i])   #list of keys in whose values temp[0] appears
	if len(indices)>1:
		for j in range(1,len(indices)):
			cluster_dictionary[indices[j]].pop()  #delete the coordinate entry of temp[0]	
			cluster_dictionary[indices[0]]=cluster_dictionary[indices[0]]+cluster_dictionary[indices[j]]	#all lists containing temp[0] are merged now, and contain temp[0] only once
		for j in range(1,len(indices)):	
			del cluster_dictionary[indices[j]]   #after merging the clusters, delete the other ones containing temp[0]

#now need to renumber the keys of cluster_dictionary from 1 to len(cluster_dictionary)
	length=len(cluster_dictionary)
	new=[] # stores the new keys from 1 to len(cluster_dictionary)
	key=cluster_dictionary.keys()   #stores the old keys from cluster_dictionary that we are about to modify
	for i in range(length):
		new.append(i+1)
	new_dict=cluster_dictionary.fromkeys(new)
	for i in range(len(key)):
		new_dict[i+1]=cluster_dictionary[key[i]]
	cluster_dictionary=new_dict

	temp.pop(0)    #pop out the value of the filled site in this iteration. In the next iteration, the next filled site will be dealt with

entire_cluster=[]  #stores the unique cluster that percolate all the 4 sides
for i in range(len(cluster_dictionary)):  #scan through all the clusters
	q_1=0   #tracks if a particular cluster touches left side
	q_2=0   #tracks if a particular cluster touches right side
	s_1=0   #tracks if a particular cluster touches bottom side
	s_2=0   #tracks if a particular cluster touches top side
	for j in range(len(cluster_dictionary[i+1])):   #scan through all the coordinates in the (i+1)th cluster
		if cluster_dictionary[i+1][j][0]==0:
			q_1+=1		# cluster touches the left side at least at one place
		if cluster_dictionary[i+1][j][0]==(n-1):
			q_2+=1          #cluster touches the right side at least at one place

	for j in range(len(cluster_dictionary[i+1])):   #scan through all the coordinates in the (i+1)th cluster
		if cluster_dictionary[i+1][j][1]==0:
			s_1+=1		#cluster touches the bottom side at least at one place
		if cluster_dictionary[i+1][j][1]==(n-1):
			s_2+=1	        #cluster touches the top side at least at one place

	if s_1!=0 and s_2!=0 and q_1!=0 and q_2!=0:   #then the percolating cluster does form
		entire_cluster=cluster_dictionary[i+1]

#print "the filled coordinates are printed below:"
#print coordinate
#print "all the clusters that are formed are printed below:"
#print cluster_dictionary
print "the lengths of all the clusters formed are given below:"
xx=[len(cluster_dictionary[cluster_dictionary.keys()[i]]) for i in range(len(cluster_dictionary))]
print xx # prints the lengths of all the clusters that are formed
print "number of grid points in each side of grid:",n
print "total number of grid points on the grid:", n**2
print "number of filled sites:",count
print "number of clusters formed:",len(cluster_dictionary)
print "number of points in the entirely percolating cluster is:",len(entire_cluster)
print "fraction of sites occupied by the percolating cluster is:",float(len(entire_cluster))/(count)
if len(entire_cluster)>0:	#if the cluster at all forms
	print "thus a percolating cluster does form"
	plot([p[0] for p in entire_cluster],[p[1] for p in entire_cluster],'r*')
	show()	
else:
	print "thus there are no spanning clusters here"
	print "but let's show the cluster which has the maximum extent,even though it doesnt percolate"	
	max_cluster=cluster_dictionary[cluster_dictionary.keys()[xx.index(max(xx))]] #the cluster with maximum extent
	plot([p[0] for p in max_cluster],[p[1] for p in max_cluster],'r*')
	print "fraction of total occupied sites covered by this largest cluster",float(len(max_cluster))/count
	show()		
time2=time()
timee=time2-time1
print timee

	


