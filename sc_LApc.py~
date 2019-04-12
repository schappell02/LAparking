import pandas as pd
import numpy as np
import datetime
from pyproj import Proj, transform
import os
from geopy.geocoders import Nominatim
import pylab as plt
from matplotlib.colors import LogNorm


def getData(file='Parking_Citations.csv',yearCutL=2015,yearCutH=2019,outF='sc_pc_2015_2018.csv'):
    '''
    Function outputs .csv file of LA parking citations with corresponding day of 
    the week, issue time, and latitude and longitude.
    As the full database provided by the city of Los Angeles is rather large,
    this is a way to retain only the necessary information.

    Inputs:
    - file [assumed to be in same directory] = path to full csv file from 
    the city of LA's website
    - yearCutL/H = input lower and upper year cut (needs to be between 2010 and 2019), set as 2015 
    and 2019 unless otherwise stated (lower limit included, upper limit not in data selection).
    - outF = path of output csv file, set to 'sc_pc_2015_2018.csv' unless otherwise stated

    Function takes full database provided by the city of LA (updated regularly on
    their website), and writes the issue day of week, time, latitude, 
    and longitude for all parking citations with properly entered latitudes and
    longitudes (about 1/7 as of Jan. 13 2019 have improperly entered coordinates).
    Will only consider citations within a certain year range.

    Lat. and long. originally given in state plane feet coordinates, function
    outputs them in degrees.
    '''

    # read larger database
    pc = pd.read_csv(file)
    # only consider parameters of interest
    use_param = ['Issue Date','Issue time','Latitude','Longitude']
    # only consider those with properly inputted lat ang long
    use_pc = pc[(pc.Latitude > 99999.0) & (pc.Latitude < 1e7)][use_param]

    # issue date column from string to datetime
    use_pc['Issue Date'] = pd.to_datetime(use_pc['Issue Date'])
    # get day of week
    use_pc['DoW'] = use_pc['Issue Date'].dt.day_name()

    # year cut
    lowt = pd.Timestamp(yearCutL,1,1)
    hight = pd.Timestamp(yearCutH,1,1)
    use_pc = use_pc[(use_pc['Issue Date'] >= lowt) & (use_pc['Issue Date'] < hight)]

    use_pc['year'] = use_pc['Issue Date'].dt.year
    # round time to nearest hour (24 hr clock)
    # example: 12:30 is 1230 in database, will be rounded to 13
    use_pc['Issue time'] = np.round(use_pc['Issue time']/100.0 + 0.2)

    inProj = Proj(init='epsg:2229', preserve_units = True)
    outProj = Proj(init='epsg:4326')

    latfe = np.array(use_pc.Latitude)
    lonfe = np.array(use_pc.Longitude)
    # transform lat and long into degrees
    use_pc.Longitude,use_pc.Latitude = transform(inProj,outProj,latfe,lonfe)

    # print to csv file
    use_pc[['Issue time','Latitude','Longitude','DoW']].to_csv(outF,index=False)




def getLaLo(searchS,uagent=None):
    '''
    Function takes input string and returns corresponding longitude and latitude.

    searchS - search string (needed input)

    uagent - user_agent string, needs to be unique and descriptive. Used for geopy. Set
    to LApc_USER_AGENT enfironmental variable unless otherwise stated.


    Function uses geopy geocoders to search for coordinates for given string. 'CA' added
    to every search. Longitude and latitude returned.
    '''
    if uagent == None:
        uagent = os.environ['LApc_USER_AGENT']
    geolocator = Nominatim(user_agent=uagent)
    location = geolocator.geocode(searchS+', CA')
    return location.longitude, location.latitude



class LAparkC():
    '''
    Class of object for plotting and calculating results for parking citations in LA.

    Calling class - give path to desired csv file (outputted using detData function).

    plotD - function that plots parking citation density for input data
    Optional input:
    PoI - point of interest, can be string (will search for longitude and latitude) or 
    coordinates (longitude and latitude).
    DoW - day of week, can be Monday through Sunday, as well as Weekday or Weekend
    ToD - time of day, accepted inputs are Early, Morning, Afternoon, and Evening. Correspond
    to 12am - 6am, 6am-noon, noon-6pm, and 6pm-midnight.
    uagent - user agent, set to environmental variables unless otherwise stated
    nbins - number of bins used for 2d histogram, 500 unless otherwise stated

    pcr_poi - function takes point of interest given and prints out parking citation rate
    at that location (citations per day per mile squared).
    PoI - needed input, can be string or list of longitude and latitude (in that order).
    uagent - user agent for PoI search, set to environmental variable unless otherwise given
    DoW - day of week, optional input (Monday through Sunday, Weekend, and Weekday are possible
    inputs).
    ToD - time of day, optional input (accepted inputs: Early, Morning, Afternoon, and Evening)
    fmil - precision level used for rate calculations (as fraction of mile), set to 0.25 miles
    unless otherwise stated
    dyrs - number of years given data covers, set to 1 year unless otherwise stated (use if only
    considering data from 2018 for example).
    '''

    def __init__(self,csvfile):
        self.all = pd.read_csv(csvfile)

    def plotD(self,PoI=None,DoW=None,ToD=None,uagent=None,nbins=500):
        parkc = self.all
        p_title = 'LA Parking Citation density'

        if DoW != None:
            if DoW == 'Weedays':
                p_title += ' during weekdays'
                parkc = parkc[(parkc.DoW != 'Saturday') & (parkc.DoW != 'Sunday')]
            elif DoW == 'Weekend':
                p_title+= ' during weekends'
                parkc = parkc[(parkc.DoW == 'Saturday') | (parkc.DoW == 'Sunday')]
            elif (DoW == 'Monday') | (DoW == 'Tuesday') | (DoW == 'Wednesday') | (DoW == 'Thursday') | (DoW == 'Friday') | (DoW == 'Saturday') | (DoW == 'Sunday'):
                parkc = parkc[parkc.DoW == DoW]
                p_title+= ' on '+DoW

        if ToD != None:
            if ToD == 'Early':
                p_title+= ' in the early morning'
                parkc = parkc[(parkc['Issue time'] >= 0) & (parkc['Issue time'] < 6)]
            elif ToD == 'Morning':
                p_title+= ' in the morning'
                parkc = parkc[(parkc['Issue time'] >= 6) & (parkc['Issue time'] < 12)]
            elif ToD == 'Afternoon':
                p_title+= ' in the afternoon'
                parkc = parkc[(parkc['Issue time'] >= 12) & (parkc['Issue time'] < 18)]
            elif ToD == 'Evening':
                p_title+= ' in the evening'
                parkc = parkc[(parkc['Issue time'] >= 18) & (parkc['Issue time'] < 24)]

        plt.hist2d(parkc.Longitude,parkc.Latitude,bins=nbins,norm=LogNorm())
        if type(PoI) == str:
            lon,lat = getLaLo(PoI,uagent=uagent)
            plt.plot(lon,lat,'r*',markersize=10)
        elif type(PoI) == list:
            plt.plot(PoI[0],PoI[1],'r*',markersize=10)
        plt.colorbar()
        plt.xlabel('Longitude')
        plt.ylabel('Latitude')
        plt.xlim([-118.7,-118.0])
        plt.ylim([33.7,34.4])
        plt.title(p_title)
        plt.show()



    def pcr_poi(self,PoI,uagent=None,DoW=None,ToD=None,fmil=0.25,dyrs=1):
        lat_to_mi = 68.92
        lon_to_mi = 57.41
        p_title = 'Ave. parking citation rate, at PoI given'

        parkc = self.all

        if type(PoI) == str:
            lon,lat = getLaLo(PoI,uagent=uagent)
        elif type(PoI) == list:
            lon,lat = PoI[0], PoI[1]


        if DoW != None:
            if DoW == 'Weedays':
                p_title += ', during weekdays'
                parkc = parkc[(parkc.DoW != 'Saturday') & (parkc.DoW != 'Sunday')]
            elif DoW == 'Weekend':
                p_title+= ', during weekends'
                parkc = parkc[(parkc.DoW == 'Saturday') | (parkc.DoW == 'Sunday')]
            elif (DoW == 'Monday') | (DoW == 'Tuesday') | (DoW == 'Wednesday') | (DoW == 'Thursday') | (DoW == 'Friday') | (DoW == 'Saturday') | (DoW == 'Sunday'):
                parkc = parkc[parkc.DoW == DoW]
                p_title+= ', on '+DoW

        if ToD != None:
            if ToD == 'Early':
                p_title+= ', in the early morning'
                parkc = parkc[(parkc['Issue time'] >= 0) & (parkc['Issue time'] < 6)]
            elif ToD == 'Morning':
                p_title+= ', in the morning'
                parkc = parkc[(parkc['Issue time'] >= 6) & (parkc['Issue time'] < 12)]
            elif ToD == 'Afternoon':
                p_title+= ', in the afternoon'
                parkc = parkc[(parkc['Issue time'] >= 12) & (parkc['Issue time'] < 18)]
            elif ToD == 'Evening':
                p_title+= ', in the evening'
                parkc = parkc[(parkc['Issue time'] >= 18) & (parkc['Issue time'] < 24)]

        xbins = np.arange(np.min(parkc.Longitude),np.max(parkc.Longitude),fmil/lon_to_mi)
        ybins = np.arange(np.min(parkc.Latitude),np.max(parkc.Latitude),fmil/lat_to_mi)
        ahist,xbins,ybins,junk = plt.hist2d(parkc.Longitude,parkc.Latitude,bins=[xbins,ybins])
        plt.close()

        xdex = np.where([xbins < lon])[1][-1]
        ydex = np.where([ybins < lat])[1][-1]
        print(p_title+' : '+str(round(ahist[xdex,ydex]/(365.0*dyrs)/fmil**2,2))+' (per day/mil^2)')
        print('Largest rate: '+str(int(np.max(ahist)/(365.0*dyrs)/fmil**2)))
