# Storage Device for Micro-manager

## MMCore API
Proposed additions to the MMCore API to accomodate the new "Storage" device. All code is pseudo C++ (without unnecessary type decorations) to improve readability. Errors are handled through exceptions (not shown).

There are three important points about this API:
- allows the acquisition engine or controlling script to be *outside* of MMCore. 
- all storage functions and any access to File System are hidden behind the MMCore API and images do not need to cross the API
- The MMCore is using the StorageDevice - a new type of MMDevice that handles reading and writing of datasets. That way it is easy to add new file formats as new devices (e.g. the same principle as adding a new camera).

The proposed API only illustrates the principle and will probably need some additional methods to be useful.

### Create Dataset
Open and close new dataset. Summary metadata passed as a "summaryMeta" parameter is in addition to implictly auto-generated metadata. Passing an empty string is fine.

acqCreateDataset returns the string handle (UUID) to the opened dataset. This handle is then used in all other api calls to refer to that dataset. Multiple datasets can be opened at the same time.

``` 
string acqCreateDataset(string path, string name, string summaryMeta);
bool acqCloseDataset(string handle);
```
After the dataset is closed, we can't add any images to it.

The summaryMetadata argument (JSON) must have a minmal set of integer parameters to define dataset:

	WIDTH
	HEIGHT
	POSITIONS
	CHANNELS
	FRAMES
	SLICES

Optionally the metadata can contain channel names, position names and any other custom metdata.

#### Note:
*We require the client to know image physical dimensions and axes extents before starting the acqusition. This might be a challenge for some use cases - see the discussion section at the end.*


### Load Dataset
This allows us to load an existing dataset and access data through the MMCore API. Loaded datasets are immutable, any attempt to add images will fail. "Loading" the dataset does not mean that it is loaded in program memory - it just means we can gain access to it through the MMCore API.

```
string acqLoadDataset(string path, string name);
```

### Delete Dataset
We can delete any dataset for which we have a valid handle.
```
void acqDeleteDataset(string handle);
```

### List Datasets
Lists all datasets in the given path.
```
vector<string> acqListDatasets(string path);
```

### Snap and Save
Performs a single image acquisition and sends the image to the Storage device. Metadata is optional, as the mandatory meta will be auto-generated.
```
void acqSnapAndSave(string handle, int frame, int channel, int slice, int position, string imageMeta);
```

### Pop and Save
Pops the next image from the queue and sends it to the Storage device. Metadata is optional, as the mandatory meta will be auto-generated.

```
void acqPopNextAndSave(string handle, int frame, int channel, int slice, int position, string imageMeta);
```
### Add external image

In order to support old style acquisition where the image makes the round-trip through the GUI we are allowing the application insert image into the storage. This can be also used to process images in the application level before adding them to the dataset.

```
void acqAddImage(string handle, int frame, int channel, int slice, int position, vector<unsigned char> pixels, string meta);
```

### Access to the data
```
string acqGetSummaryMeta(string handle);
string acqGetImageMeta(string handle, int frame, int channel, int slice, int position);
BINARYDATA acqGetImagePixels(string& handle, int frame, int channel, int slice, int position, int size);
```

## Storage Device API
To be determined after the MMCore API is complete. The device API will mostly mirror the MMCore API, plus some calls to insert pixel data. MMCore will need to properly manage this device, e.g. wire its internal image buffers to it.

Storage Device API is not visible to the application (user-client) so we will treat it as an MMCore implementation detail.

## Conceptual Block Diagram
![alt text](https://github.com/go2scope/MM-storage-api/blob/main/mm-python-app-2024-02-15-1225.png)


# Discussion
Nico's comments:
* The API assumens 6D image model, maybe we should generalize to variable N dimensional model
* The API assigns physical meaning to image dimension coordinates, maybe we can generalize to abstract dimensions. The interpretation of dimensions is then assigned by the acquisition engine
* Consider supporting variable image dimensions during acquisition
* Consider the case where we have multiple cameras acuiring simultaneously. Not clear if the current API supports that case.

