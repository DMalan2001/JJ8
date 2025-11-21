import maya.cmds as mc
if mc.window('Ribbon_Maker', ex=True):
    mc.deleteUI('Ribbon_Maker')
mc.window('Ribbon_Maker')
mc.columnLayout(adj=True)
mc.text('RIB', l='Ribbon Setup', w= 100)
mc.separator(style='single',height=10)
mc.button(l='MakeCurves',c='curveToEdge()')
mc.button(l='Make Ribbon', c='makeLoft()')
mc.button(l='Reverse Normal', c='reverse()')
ribName=mc.textFieldGrp(l='Ribbon Name:')
mc.button(l='Rename Ribbon',c='RibSurf()')
mc.button(l='Select Attribute Controller and Make Deformers', c='blendShapeRib()')
mc.separator(style='single',height=10)
mc.text('Build', l='Building Ribbon and Orientation', w= 100)
mc.separator(style='single',height=10)
mc.button(l='Build Ribbon', c='buildRibbon()')
mc.button(l='Freeze Orientation', c= 'frzOriTF()')
mc.button(l='Make Controller Joints', c='makeCtrlJnt()')
mc.separator(style='single',height=10)
mc.text('FK', l= 'FK Setup')
mc.separator(style='single',height=10)
mc.button(l='FK Hybrid', c='fkRibbon()')
mc.button(l='Freeze Orientation', c= 'frzOriTF()')
mc.button(l='FK Ctrl Jnt', c= 'fkGroup()')
mc.separator(style='single',height=10)
mc.text('Skinning',l= 'Select For Skinning')
mc.separator(style='single',height=10)
mc.button(l='Select Ctrl Joints',c='selectCtrlJnt()')
mc.button(l='Select Bind Joints', c='selectBindJnt()')
mc.showWindow('Ribbon_Maker')

#Sets Name of Ribbon
def RibSurf():
    if mc.ls(sl=True):
        rn=mc.textFieldGrp(ribName, q=True, tx=True)
        name=mc.rename(rn+'_Ribbon_Main')
        mc.xform(name, cp=True)

#Makes Curves to Loft
def curveToEdge():
    mc.polyToCurve(form=2, degree=1, conformToSmoothMeshPreview=1)

#Joining Curves to make Ribbon
def makeLoft():
    mc.select(cl=True)
    mc.select('polyToCurve1', 'polyToCurve2')
    curveList=mc.ls(sl=True)
    rib=mc.loft(curveList)
    mc.xform(rib, cp=True)
    mc.delete(curveList)

#Reverse Ribbon Normal
def reverse():
    mc.reverseSurface(ch=False)

#bulid follicles and jnts
def buildRibbon():
    rn=mc.textFieldGrp(ribName, q=True, tx=True)
    patchesU=mc.getAttr(rn+'_Ribbon_Main'+'.spansUV.spansU')
    patchesV=mc.getAttr(rn+'_Ribbon_Main'+'.spansUV.spansV')
    u=patchesU + 1
    v=patchesV + 1
    if patchesU>patchesV:
        for i in range(u):
            mc.createNode('pointOnSurfaceInfo',n=rn+'posi')
            posiList=mc.ls(rn+'posi*',sl=True)[0]
            mc.connectAttr(rn+'_Ribbon_MainShape.worldSpace', posiList+'.inputSurface', force=True)
            folli=mc.createNode('follicle',n=rn+'follicle'+str(i+1))
            mc.pickWalk(d='up')
            mc.rename(rn+'_fol'+str(i+1))
            folliList=mc.listRelatives(folli, ap=True)[0]
            mc.connectAttr(posiList+'.position',folliList+'.translate')
            mc.setAttr(posiList+'.parameterV',(0.5))
            mc.setAttr(posiList+'.parameterU',(i/(u-1)))
            bj=mc.joint(n=rn+'_Bind+Jnt'+str(i+1),rad=(.3))
        folGrp=mc.ls(rn+'_fol*')
        mc.group(folGrp,n=rn+'_Follicle_Grp')
    else:
        for i in range(v):
            mc.createNode('pointOnSurfaceInfo',n=rn+'posi')
            posiList=mc.ls(rn+'posi*',sl=True)[0]
            mc.connectAttr(rn+'_Ribbon_MainShape.worldSpace', posiList+'.inputSurface', force=True)
            folli=mc.createNode('follicle',n=rn+'follicle'+str(i+1))
            mc.pickWalk(d='up')
            mc.rename(rn+'_fol'+str(i+1))
            folliList=mc.listRelatives(folli, ap=True)[0]
            mc.connectAttr(posiList+'.position',folliList+'.translate')
            mc.setAttr(posiList+'.parameterV',(i/(v-1)))
            mc.setAttr(posiList+'.parameterU',(0.5))
            bj=mc.joint(n=rn+'_Bind_Jnt'+str(i+1),rad=(.3))
        folGrp=mc.ls(rn+'_fol*')
        mc.group(folGrp,n=rn+'_Follicle_Grp')

#Making Deformers 
def blendShapeRib():
    rn=mc.textFieldGrp(ribName, q=True, tx=True)
    sineRib= mc.duplicate(rn+'_Ribbon_Main', n=rn+'_Ribbon_Sine')
    twistRib= mc.duplicate(rn+'_Ribbon_Main', n=rn+'_Ribbon_Twist')
    sel=mc.ls(sl=True)[0]
    if mc.ls(sl=True):
        mc.addAttr(sel,longName='sAttr',niceName='--------',at='enum',en='Sine Attr',keyable=True)
        mc.addAttr(sel,longName='Amplitude',at='float',keyable=True)
        mc.addAttr(sel,longName='Wavelength',at='float',keyable=True)
        mc.addAttr(sel,longName='Sine_Offset',at='float',keyable=True)
        mc.addAttr(sel,longName='DropOff',at='float',keyable=True,max=1, min=-1 )
        mc.addAttr(sel,longName='LowBound', at='float',min=-1, max=0, dv=-1, keyable=True)
        mc.addAttr(sel,longName='HighBound', at='float',max=1, min=0, dv=1, keyable=True)
        mc.addAttr(sel,longName='Sine_Twist', at='float', keyable=True)
        mc.addAttr(sel,longName='tAttr',niceName='--------',at='enum',en='Twist Attr',keyable=True)
        mc.addAttr(sel,longName='Twist_Start_Angle',at='float',keyable=True)
        mc.addAttr(sel,longName='Twist_End_Angle',at='float',keyable=True)
    mc.select(cl=True)
    BShape=mc.blendShape(sineRib,twistRib,rn+'_Ribbon_Main')
    l=mc.rename(BShape,rn+'_BS')
    mc.select(sineRib)
    sineDef=mc.nonLinear(type='Sine', n=rn+'Sine_Def')
    mc.select(cl=True)
    mc.select(twistRib)
    TwistDef=mc.nonLinear(type='Twist', n=rn+'Twist_Def')
    AttrCtrl=mc.select(sel)
    mc.connectAttr(sel+'.Amplitude',rn+'Sine_Def.amplitude')
    mc.connectAttr(sel+'.Wavelength',rn+'Sine_Def.wavelength')
    mc.connectAttr(sel+'.Sine_Offset',rn+'Sine_Def.offset')
    mc.connectAttr(sel+'.HighBound',rn+'Sine_Def.highBound')
    mc.connectAttr(sel+'.LowBound',rn+'Sine_Def.lowBound')
    mc.connectAttr(sel+'.DropOff',rn+'Sine_Def.dropoff')
    mc.connectAttr(sel+'.Sine_Twist',rn+'Sine_DefHandle.rotateY')
    mc.connectAttr(sel+'.Twist_Start_Angle',rn+'Twist_Def.startAngle')
    mc.connectAttr(sel+'.Twist_End_Angle',rn+'Twist_Def.endAngle')
    mc.group(n=rn+'_Attr_Grp',em=True)
    mc.parent(rn+'_Ribbon_Sine',rn+'_Ribbon_Twist',rn+'Sine_DefHandle',rn+'Twist_DefHandle',rn+'_Attr_Grp')

#Freeze Orientation
def frzOriTF():
    rn=mc.textFieldGrp(ribName, q=True, tx=True)
    bList=mc.ls(rn+'_Bind_Jnt*')
    mc.select(bList)
    mc.makeIdentity(apply=True, r=True)
    
#makeCopy of selected Joints and thier Group offset
def makeCtrlJnt():
    rn=mc.textFieldGrp(ribName, q=True, tx=True)
    selection=mc.ls(sl=True)
    each=[0]
    for each in selection:
        mc.select(each,r=True)
        mc.joint()
        list=mc.ls(sl=True)[0]
        ctrlJnt=mc.rename(str(rn)+'_Ctrl_Jnt')
        ctrlGrp=mc.group(n=rn+'_Ctrl_Grp',em=True)
        mc.matchTransform(ctrlGrp, ctrlJnt)
        mc.parent(ctrlJnt, ctrlGrp)
    mCtrlGrp=mc.group(n=rn+'_Master_Ctrl_Grp', em=True)
    CtrlGrp=mc.ls(rn+'_Ctrl_Grp*')
    mc.parent(CtrlGrp,mCtrlGrp)
    
#build Fk Ribbon Hybrid
def fkRibbon():
    rn=mc.textFieldGrp(ribName, q=True, tx=True)
    patchesU=mc.getAttr(rn+'_Ribbon_Main'+'.spansUV.spansU')
    patchesV=mc.getAttr(rn+'_Ribbon_Main'+'.spansUV.spansV')
    u=patchesU + 1
    v=patchesV +1
    if patchesU>patchesV:
        for i in range(u):
            mc.createNode('pointOnSurfaceInfo',n=rn+'posi')
            posiList=mc.ls(rn+'posi*',sl=True)[0]
            mc.connectAttr(rn+'_Ribbon_MainShape.worldSpace', posiList+'.inputSurface', force=True)
            folli=mc.createNode('follicle',n=rn+'follicle'+str(i+1))
            mc.pickWalk(d='up')
            mc.rename(rn+'_fol'+str(i+1))
            folliList=mc.listRelatives(folli, ap=True)[0]
            mc.connectAttr(posiList+'.position',folliList+'.translate')
            mc.setAttr(posiList+'.parameterV',.5)
            mc.setAttr(posiList+'.parameterU',(i/(u-1)))
            mc.joint(n=rn+'_Bind_Jnt'+str(i+1), rad=(.3))
        folGrp=mc.ls(rn+'_fol*')
        mc.group(folGrp,n=rn+'_Follicle_Grp')
    if patchesV>patchesU:
        for i in range(v):
            mc.createNode('pointOnSurfaceInfo',n=rn+'posi')
            posiList=mc.ls(rn+'posi*',sl=True)[0]
            mc.connectAttr(rn+'_Ribbon_MainShape.worldSpace', posiList+'.inputSurface', force=True)
            folli=mc.createNode('follicle',n=rn+'follicle'+str(i+1))
            mc.pickWalk(d='up')
            mc.rename(rn+'_fol'+str(i+1))
            folliList=mc.listRelatives(folli, ap=True)[0]
            mc.connectAttr(posiList+'.position',folliList+'.translate')
            mc.setAttr(posiList+'.parameterV',(i/(v-1)))
            mc.setAttr(posiList+'.parameterU',(0.5))
            bj=mc.joint(n=rn+'_Bind_Jnt'+str(i+1),rad=(.3))
        folGrp=mc.ls(rn+'_fol*')
        mc.group(folGrp,n=rn+'_Follicle_Grp')

#Making FK Ctrlsr
def fkGroup():
    rn=mc.textFieldGrp(ribName, q=True, tx=True)
    selection=mc.ls(sl=True)
    each=[0]
    for each in selection:
        sel=mc.select(each,r=True)
        mc.duplicate(n=rn+'_Ctrl_Jnt')
    ctrlGrp=mc.group(n=rn+'_Ctrl_Grp',em=True)
    mc.matchTransform(ctrlGrp,rn+'_Ctrl_Jnt')
    for i in range(len(selection)-1):
        list1=mc.ls(rn+'_Ctrl_Jnt*')[i]
        list2=mc.ls(rn+'_Ctrl_Jnt*')[i+1]
        mc.parent(list2,list1)
    mc.parent(rn+'_Ctrl_Jnt',ctrlGrp)

#Select CtrlJnts to bind skin on Ribbon
def selectCtrlJnt():
    rn=mc.textFieldGrp(ribName, q=True, tx=True)
    allCtrl=mc.ls(rn+'_Ctrl_Jnt*')
    mc.select(allCtrl)
    
#Select BindJnts to bind skin
def selectBindJnt():
    rn=mc.textFieldGrp(ribName, q=True, tx=True)
    allBinds=mc.ls(rn+'_Bind_Jnt*')
    mc.select(allBinds, add=True)
