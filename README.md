describe('processNameAndAddress', () => {

  it('should process identifiers when dataObjList is not an array', () => {
    const dataMapModel = {
      identifiers: ['employerName', 'address']
    };
    const taskData = {};
    const dataObjList = {
      employerName: 'John Doe',
      address: '123 Main St'
    };

    spyOn(service, 'getObjectValue').and.callThrough();
    spyOn(service, 'processAddress').and.callThrough();

    service.processNameAndAddress(taskData, dataMapModel);

    expect(service.getObjectValue).toHaveBeenCalledWith(dataObjList, 'employerName');
    expect(service.getObjectValue).toHaveBeenCalledWith(dataObjList, 'address');
    expect(service.processAddress).toHaveBeenCalledWith('123 Main St');
  });

  it('should push identifierValue if it is not empty', () => {
    const dataMapModel = {
      identifiers: ['employerName']
    };
    const taskData = {};
    const dataObjList = {
      employerName: 'John Doe'
    };

    spyOn(service, 'getObjectValue').and.returnValue('John Doe');

    const result = service.processNameAndAddress(taskData, dataMapModel);

    expect(result).toContain('John Doe');
  });

  it('should process address if identifierName is of type ADDRESS', () => {
    const dataMapModel = {
      identifiers: ['location.address']
    };
    const taskData = {};
    const dataObjList = {
      'location.address': '456 Elm St'
    };

    spyOn(service, 'getObjectValue').and.returnValue('456 Elm St');
    spyOn(service, 'processAddress').and.returnValue('Processed Address');

    const result = service.processNameAndAddress(taskData, dataMapModel);

    expect(service.processAddress).toHaveBeenCalledWith('456 Elm St');
    expect(result).toContain('Processed Address');
  });

});
