# ISSUE
: error: cannot convert 'MainWindow*' to 'const QTimer&amp;


#include "mainwindow.h"
#include <iostream>
#include <sstream>
#include <QDebug>
#include<Windows.h>
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{

    bMarkROI = false;

    //初始化谱数据信息
    this->initSpectrum();
    //DP5Protocol = new Protocol;

    mainUI = new UITopWidget(this);
    usbObject = new usbIODevice();
    usbThread = new QThread;
    calibrate = new Calibrate(spectrum,this);
    peakSearch = new PeakSearch(spectrum,this);
    defineROI = new DefineROI(spectrum,this);
    openStart = new OpenStart(spectrum,this);



    sfInfo.m_iNumChan=4096;
    sfInfo.strTag="live_data";
    sROI = NULL;

    this->setCentralWidget(mainUI);
    this->createMenuBarActions();
    initWindget();
    //openStart->show();//
    //openStart->exec();
    connect(mainUI->rightButton, &QPushButton::clicked, this, &MainWindow::onXZoomIn);
    connect(mainUI->leftButton, &QPushButton::clicked, this, &MainWindow::onXZoomOut);
    connect(mainUI->upButton, &QPushButton::clicked, this, &MainWindow::onYZoomIn);
    connect(mainUI->downButton, &QPushButton::clicked, this, &MainWindow::onYZoomOut);

    timer15ms = new QTimer(this);
  //QTimer *timer15ms = new QTimer(this);
    timer15ms->setInterval(1000);//1s
    //connect(timer15ms, SIGNAL(timeout()), this, SLOT(slot_refreshParameterPanelStatus()));
    //修改
    connect(timer15ms, &QTimer::timeout, this, &MainWindow::slot_refreshParameterPanelStatus);

    timer2ms = new QTimer(this);
    timer2ms->setInterval(1000);//1s
    connect(timer2ms, SIGNAL(timeout()), this, SLOT(drawSpectrumPlot()));

    connect(mainUI->spectrScrollBar, &QAbstractSlider::valueChanged, this, &MainWindow::setXAxisRange);
    connect(mainUI->spectrPlot->xAxis, SIGNAL(rangeChanged(QCPRange)), this, SLOT(setScrollBarPage()));
    connect(mainUI->spectrPlot,&MyQCustomPlot::signal_mouseEvent,this,&MainWindow::slot_getMousePos);
    connect(mainUI->spectrPlot, &QCustomPlot::selectionChangedByUser, this, &MainWindow::selectGraph);

    timer15ms->start();
    timer2ms->start();

    //2021/08/30 version 1.3
    connect(CalibrateSwitch, &QAction::triggered, this, &MainWindow::onCalibrateSwitch);



    connect(this,&MainWindow::sig_calibrate,calibrate,&Calibrate::changeCalibrate);

    connect(peakSearch,&PeakSearch::sig_AddOrReplaceROI,defineROI,&DefineROI::slot_AddOrReplaceROIList);
    connect(peakSearch,&PeakSearch::sig_drawPeakROISpectrum, this,&MainWindow::slot_drawROISpectrum);
    connect(peakSearch,&PeakSearch::sig_OkClicked,defineROI,&DefineROI::onOkButton);
    connect(peakSearch,&PeakSearch::sig_CancelClicked,defineROI,&DefineROI::onCancelButton);
    connect(peakSearch->ui->DetailsBt,&QAbstractButton::clicked,defineROI,&DefineROI::onDetailsButton);


    connect(this, &MainWindow::sig_addROI, defineROI, &DefineROI::slot_addROI);
    connect(this, &MainWindow::sig_setChannel, defineROI, &DefineROI::getROICentroid);
    connect(this, &MainWindow::sig_setChannel, calibrate, &Calibrate::setChannel);



    connect(defineROI,&DefineROI::sig_setROICentroid,calibrate,&Calibrate::getCentroid);
   // connect(defineROI,&DefineROI::sig_drawROISpectrum,this,&MainWindow::drawROISpectrum);
    connect(defineROI,&DefineROI::sig_drawROISpectrum,this,&MainWindow::slot_drawROISpectrum);
    connect(defineROI,&DefineROI::sig_setDetails,this,&MainWindow::slot_setQAcROIDetails);
    connect(defineROI,&DefineROI::sig_setDetails,peakSearch,&PeakSearch::slot_setDetailsBt);

    connect(calibrate,&Calibrate::sig_setCalibrate,this,&MainWindow::slot_setCalibrate);

    //2021/09/16
    connect(openStart,&OpenStart::sig_ParsePacket ,this,&MainWindow::ReceiveData);
    connect(openStart,&OpenStart::sig_setAxisX ,this,&MainWindow::setAxisX);



}
